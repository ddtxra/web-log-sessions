# Web Sessions computed using Web Server Logs 

(This project is under development)

The aim of this project is to compare Google Analytics Sessions or Piwik Visits with sessions that could be computed analysing web server logs (like Apache logs).

From Google Analytics definition: A session is a group of interactions that take place on your website within a given time frame. A single user can open multiple sessions. By default on GA, a sessions ends when:
* After 30 minutes of inactivity (this value can be changed, but by default it's 30minutes)
* At midnight

Source: https://support.google.com/analytics/answer/2731565?hl=en (There are other parameters related to campaigns, but we don't take this into account)

Piwik has a similar definition for visit: http://piwik.org/faq/general/faq_36/

# Install

```shell
> sbt assembly
```
This creates a web-log-sessions.jar inside the target folder.

# Run

```shell
java -jar target/web-log-sessions.jar
```

You can also add some options:

```shell
java -jar -DlogFilesFolder=../logs/ -DlogFilesFolder="^access_log-15121\d{7}$" target/web-log-sessions.jar
```

* logsFolder = The folder where to find log giles
* filesRegex = The regex used to filter the files inside the folder
* sessionTimeoutInMinutes = The time of a session in minutes (default = 30)
* filterBots = Defines if bots are filtered (default false)

* botsRegex = TODO default = "bot|spider|facebook|feed|crawler|baidu|bing|yahoo|googletoolbar"
* fileFormatRegex = TODO default is: 
* excludeRequestsRegex = TODO 
* excludeIPsRegex = TODO 

* computeTopAgents = Defines if bots are filtered (default false)
* computeTopRequests = Defines if bots are filtered (default false)
* computeTopIps = Compute top IPs


# Results

"Normal" HTML website
![alt text](assets/ga-vs-log-html.png "Normal HTML application")

"Angular" website (single page application with partials)
![alt text](assets/ga-vs-log-spa.png "Single Page Application")


## Sessions advantages

Measuring sessions (aka Piwik visits) has some advantages compared to hits when one wants to measure website **usage**:

* It is **not dependent on the technology or the design** used to build the website. For example a SPA (Single Page Application) in AngularJS may have many partials (html files) to build one single page. It can potentially generate plenty of hits on the server side, but will generate only one session (which is more correct in term of usage). 
On the other hand, if a site is build with one basic HTML file, only one hit will be generated on the server side. 
Those websites have the same "usage", but the hits can differ a lot, while the number of sessions are the same.

* Other **dependent resources** used to build the pages like images, css/js files, ... won't biase the number of sessions, because they all are part of the same session. 
But they will surely biase hits, and it is sometimes difficult to agree what is a page and what is not (only looking at the logs).

* It increases if **unique IPs** increases. 
In term of usages it's better to have 2 users acccessing the same resource, rather than 1 user accessing 2 pages. The number of hits won't reflect this information.

* It is much less affected by **monitoring tools** / **crawlers** or even **scripting / programmatic access**. 
For example if a monitoring tool, access the website every minute, the number of hits will increase while the session will only be 1 at the end of the day (indexing crawlers should be removed, but scription or programmatic access, like wget, should be counted).

* It's less affected by the **cache** configuration. 
If a cache (based on expiration time) of 5minutes is defined and session is set to 30minutes sliding time window, then the number of sessions will be the same. While the number of hits can decrease a lot.

* It's in not very much affected by a **small downtime**  
(less than 30 min) of the service, while the number of hits can be. Small downtimes of the service should not be reflected on the usage statistics (other metric should be used to count that).

* It is not dependent on the **number of pages / urls** or **interactivity** the resource offers. 
For example if a resource contains 10 entries and for each entry creates 10 sections (urls) to be more interactive, while another resource present these same entries in just 10 different pages, the number of sessions may be the same while the number of hits may differ a lot. 

As well as hits, it captures other data access like txt, xml, json data access (that are not caugh by Google Analytics)

## Limitations / Considerations

Note that this is not an exact science. It is not possible to reproduce exactly the same numbers using client and server side technology. Google Analytics relies on browser cookies and this alternative relies on server logs. 

Take for example the following scenario:
Two persons with the same version of a browser, going to the same website at the same moment in time and sitting behind an institution that only exposes one single IP address to the public world: The web log sessions won't be able to tell that the requests are coming from two different persons, while  Google Analytics that relies on browser cookies will be able to tell that the requests are coming from two persons and therefore will create two sessions.

On the other hand, if someone using a programmatic tool that does not interpret javascript (such as wget) and downloading bunches of files from a website (such as  XML files). Google analytics won't be able to intercept those requests and will never count any session associated to this, while server log sessions will.

Nevertheless, the results have shown so far that the approximation is quite satisfactory.
