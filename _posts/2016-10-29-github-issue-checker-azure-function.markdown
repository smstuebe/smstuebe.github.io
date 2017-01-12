---
layout:     post
title:      "Validating new github issues with Azure Functions"
subtitle:   "Using Azure Functions and github webhooks to check if the issue template to relieve some pain of frustrated repository owners."
date:       2017-01-12 12:00:00
author:     "Sven-Michael Stübe"
header-img: "img/home-bg.jpg"
tags: [Azure Functions, github, issues, issue template, webhook]
---

In 2016 github added a feature, that allows users to create templates for new issues and pull requests. This is nice, because if the maintainers can help users to provide all necessary data without asking for them several times.
Unfortunately, some people just delete all of the template and start typing. In this blog post I'll show my solution for this problem.

<h2 class="section-heading">Functionality</h2>

<img src="/img/issue-checker-system.png" />

When a user opens a new issue, github sends the issue information to a Azure Function web hook. The function analyses the issue data and compares the issue text with the issue template. After computing the matching quote, it adds an comment using the github API.

<h2 class="section-heading">Setup</h2>
<h3 class="section-heading">Azure Function</h3>
<h4 class="section-heading">Creation</h4>
Creating a new Azure Function is very easy.

- open <a href="https://portal.azure.com/" target="_blank">https://portal.azure.com/</a>
- add a Function app and open it
- click `New Function`
- select scenario `Webhook + API`
- select language `C#`
- click `Create this function`

If you need a more detailed tutorial, click <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-a-web-hook-or-api-function" target="_blank">here</a>.

<h4 class="section-heading">Configuration</h4>
The github API for .NET is implemented in `Octokit` and it's available on NuGet.
To add the NuGet package to your Function:

- click `View files`
- click `Add`
- name it `project.json`
- paste the following code and save it


{% highlight json %}
{
    "frameworks": {
        "net46": {
            "dependencies": {
                "Octokit": "0.23.0"
            }
        }
    }
}
{% endhighlight %}


<h3 class="section-heading">Github</h3>

<h4 class="section-heading">Webhook</h4>
Now, you have to configure github to call your webhook when a new issue was opened.

- Open the settings of the repository that you want to monitor.
- click `Webhooks`
- click `Add Webhook`
- enter the Azure Function url into `Payload URL`
- enter the GitHub Secret into `Secret`
- select `Let me select individual events`
- select `Issues`

<img src="/img/issue-checker-webhook.png" />

To verify your setup, you can open an issue. If you look at the log output of your function, you'll see something like:
{% highlight text %}
2017-01-11T22:22:42.222 C# HTTP trigger function processed a request.
{% endhighlight %}

**Pro tip:** Before opening the issue, add this line to your function:

{% highlight c# %}
log.Info($"Data: {data}");
{% endhighlight %}

The request body will be written to the log. You can then copy it and paste it into the `Test` section and every time you press `Run` the "real" request will be processed. So you don't need to open a new issue on github to test you function.

<h4 class="section-heading">OAuth API token</h4>
The github API requires an access token. Create a new one following these steps:

- open <a href="https://github.com/" target="_blank">https://github.com/</a>
- log in and open your settings
- click `Personal access tokens`
- click `Generate new token`
- enter a name and select `public_repo`
- click `Generate token`
- save the generated token

<img src="/img/issue-checker-token.png" />

<h4 class="section-heading">Issue template</h4>
If your repository doesn't contain an issue template `ISSUE_TEMPLATE.md`, add one. Click <a href="https://github.com/blog/2111-issue-and-pull-request-templates" target="_blank">here</a> for more information on issue templates.
Additionally add the file `ISSUE_TEMPLATE_CHECK.md`. This file contains the lines that should be included in the issue in the given order. I've chosen the headings and some bold bullet points.

{% highlight markdown %}
## Steps to reproduce
## Expected behavior
## Actual behavior
### Crashlog
## Configuration
**Version of the Plugin:**
**Platform:**
**Device:**
{% endhighlight %}


<h2 class="section-heading">Let's code!</h2>

Now, everything is setup correctly and you can code (or copy paste) your Azure Function. The code is available on <i class="fa fa-github"></i><a href="https://github.com/smstuebe/github-issue-template-checker" target="_blank">github</a>.

{% highlight c# %}
public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    dynamic data = await req.Content.ReadAsAsync<object>();
    log.Info($"Data. {data}");
    await ProcessIssueAsync(data);  
   
    return req.CreateResponse(HttpStatusCode.OK);
}

private static async Task ProcessIssueAsync(dynamic data)
{
    if(data?.action != "opened")
        return;

    var owner = (string)data.repository.owner.login;
    var repository = (string)data.repository.name;    
    var repositoryId = (long)data.repository.id;
    var branch = (string)data.repository.default_branch; 

    var issueLines = GetLines((string)data.issue.body);
    var templateLines = (await GetTemplateElements(owner, repository, branch)).ToArray();

    var matchingQuote = CheckIssueWithTemplate(issueLines, templateLines);
    var message = GetMessage("smstuebe", matchingQuote);
    await CreateCommentAsync(repositoryId, (int)data.issue.number, message);
}
{% endhighlight %}

First, call `ProcessIssueAsync` from your `Run` function. `ProcessIssueAsync` implements the workflow of the issue. You need to check if the `action` is `"opened"`, because github sends all issue related events to your webhook.
The function then:

- reads some properties from the request
- splits the issue text into lines
- gets all lines of `ISSUE_TEMPLATE_CHECK.md`
- checks the `issueLines` using the `templateLines` and calculates a matching quote in percent
- generates a message based on the matching quote
- creates a new comment on the issue

The format of the issue request is documented <a href="https://developer.github.com/v3/activity/events/types/#issuesevent" target="_blank">here</a>.

**CheckIssueWithTemplate** 

`CheckIssueWithTemplate` counts the lines of `templateLines` that occur in `issueLines` and calculates a quote that expresses how well the check condition was satisfied. You can implement any logic you want. It just has to project two string arrays to a number between `0` (=issue template wasn't used at all) and `1` (=100% sure that the issue template was used).

{% highlight c# %}
private static double CheckIssueWithTemplate(string[] issueLines, string[] templateLines)
{
    if (templateLines.Length == 0)
        return 1;

    var templateList = templateLines.ToList();
    foreach (var issueLine in issueLines)
    {
        var found = templateList.FirstOrDefault(tpl => issueLine.StartsWith(tpl));

        if (!string.IsNullOrEmpty(found))
        {
            templateList.Remove(found);
        }
    }
    var matches = templateLines.Length - templateList.Count;
    return matches / (double)templateLines.Length;
}
{% endhighlight %}

**GetMessage** 

The `GetMessage` generates a praisingly message if the user used the template and a sad one if he probably didn't. You can add your on text and thresholds if you want.
{% highlight c# %}
static string GetMessage(string userName, double matchingQuote)
{
    string message;
    if (matchingQuote > 0.9)
    {
        message = "Thanks for using the issue template :kissing_heart:\n" +
                    "I appreciate it very much. I'm sure, the maintainers of this repository will answer, soon.";
    }
    else
    {
        message = $"It seems like ({(1 - matchingQuote):P}) you haven't used our issue template :cry: " +
                    $"I think it is very frustrating for the repository owners, if you ignore them.\n\n" +
                    $"If you think it's fine to make an exception, just ignore this message.\n" +
                    $"**But if you think it was a mistake to delete the template, please close the issue and create a new one.**\n\n" +
                    $"Thanks!";
    }                       

    return $"Hi @{userName},\n\n" +
            $"I'm the friendly issue checker.\n" +
            message;
}
{% endhighlight %}

**CreateCommentAsync**
Last you have just to send the comment to github. The API makes this very easy. The developers  of github (and ofc. the contributers) have done a great job!
You have to replace the token with your own one.

{% highlight c# %}
private static async Task CreateCommentAsync(long repositoryId, int issueNumber, string message)
{
    var client = new GitHubClient(new ProductHeaderValue("github-issue-checker"));
    var tokenAuth = new Credentials("<github OAuth token>");
    client.Credentials = tokenAuth;
    var comment = await client.Issue.Comment.Create(repositoryId, issueNumber, message);
}

{% endhighlight %}


<h2 class="section-heading">Result</h2>

<img src="/img/issue-checker-create-issue.png" />
will result in:
<img src="/img/issue-checker-bad.png" />

A nice looking issue will be commented like:
<img src="/img/issue-checker-good.png" /> 

<h2 class="section-heading">Possible additions</h2>
I think you are now aware of the possibilities that the connection of github webhooks, Azure Functions and the github API offers you. The Function can be easily enhanced with features like

- close issues automatically if match qoute is smaller than 10%
- save issue opener to database and block them after 3 bad issues
- make the matching algorithm `CheckIssueWithTemplate` more intelligent
- make the messages configurable like `ISSUE_TEMPLATE_CHECK.md` 
- ...

<i class="fa fa-twitter"></i> <a href="https://twitter.com/intent/tweet?text=@stuebe2k7" target="_blank">Tweet</a> me your best idea and win a like :) 

<small>Found a typo? Send me a pull request!</small>
