# Live Project - Django App - NHL Hockey Statistic Tracker
Repository for code snippets and a description of work I accomplished during my 1st Tech Academy Live Project. I worked on bulding a single Django application within a larger group project, and the focus of my application was creating a place for user's to store data about local hockey player, or view stats and highlights for their favorite NHL teams and players using the NHL.com API.

## Table of contents:
### Front End:
* I did not include any of the CSS code snippets in this file, but the screenshots throughout display the general styling
* [Javascript Table Sorting](https://github.com/chasetmartin/Live_Project_1/blob/main/README.md#javascript-function-to-sort-tables-alphabetically-when-displayed-in-the-application)
### Back End:
* [Creating Django Model](https://github.com/chasetmartin/Live_Project_1/blob/main/README.md#creating-a-django-model-and-model-manager)
* [Create, Read, Edit, Delete View Functions](https://github.com/chasetmartin/Live_Project_1/blob/main/README.md#writing-view-functions-to-create-read-edit-and-delete-instances-of-my-hockey-player-model)
* [Accessing the NHL.com API](https://github.com/chasetmartin/Live_Project_1/blob/main/README.md#accessing-the-nhlcom-api)
     * [Requesting All NHL Teams](https://github.com/chasetmartin/Live_Project_1/blob/main/README.md#the-view-function-to-request-a-list-of-all-nhl-teams-from-the-api)
     * [Requesting Roster & Most Recent Game Info](https://github.com/chasetmartin/Live_Project_1/blob/main/README.md#the-view-function-api-request-for-the-roster-most-recent-game-date-score-and-primary-key-for-the-team-selected-by-the-user)
     * [Requesting Player Stats and Saving to Model](https://github.com/chasetmartin/Live_Project_1/blob/main/README.md#the-view-function-api-request-for-the-selected-players-stats-with-the-code-to-save-the-stats-to-the-hocky-player-database-model)
     * [Requesting URL for Video Highlights](https://github.com/chasetmartin/Live_Project_1/blob/main/README.md#the-view-function-api-request-for-the-url-associated-with-a-teams-most-recent-game-video-highlights)

<br>

## Creating the basic templates and home page view:
<img width="1423" alt="Screen Shot 2023-01-19 at 9 52 45 AM" src="https://user-images.githubusercontent.com/36861079/213522379-1c75ebb8-2496-41bf-a271-c0d0b28e6cf4.png">

<br>

## Creating a Django Model and Model Manager:
This model was designed to work with both user-created Hockey Player objects, or NHL players retreived from the NHL.com API and saved by the user. So the fields had to make sense for either type of entry.
```
class HockeyPlayer(models.Model):
    name = models.CharField(max_length=50, default="", blank=True, null=False)
    timeonice = models.CharField(max_length=50, default="", blank=True, null=False)
    points = models.IntegerField(default="", blank=True)
    goals = models.IntegerField(default="", blank=True)
    assists = models.IntegerField(default="", blank=True)
    hits = models.IntegerField(default="", blank=True)
    faceoffpct = models.DecimalField(default="", blank=True, decimal_places=2, max_digits=5)
    plusminus = models.IntegerField(default="", blank=True)
    date_created = models.DateTimeField(auto_now=False, auto_now_add=True)

    HockeyPlayers = models.Manager()

    def __str__(self):
        return self.name
```

<br>

## Writing view functions to create, read, edit, and delete instances of my Hockey Player Model:
### Create model and render create entry page
```
def hockeytracker_create(request):
    form = HockeyPlayerForm(data=request.POST or None)
    if request.method == 'POST':
        if form.is_valid():
            form.save()
            return redirect('hockeytracker_read')
        else:
            print(form.errors)
    content = {'form': form}
    return render(request, 'HockeyTracker/hockeytracker_create.html', content)
```
### Display all items from database
```
def hockeytracker_read(request):
    hockeyplayer = HockeyPlayer.HockeyPlayers.all()
    content = {'hockeyplayer': hockeyplayer}
    return render(request, 'HockeyTracker/hockeytracker_read.html', content)
```
<img width="1421" alt="Screen Shot 2023-01-19 at 9 50 14 AM" src="https://user-images.githubusercontent.com/36861079/213521751-cee2d414-ddf3-4a43-8852-f629bc9010f8.png">

### Details Page for selected player
```
def hockeytracker_details(request, pk):
    hockeyplayer = get_object_or_404(HockeyPlayer, pk=pk)
    content = {'hockeyplayer': hockeyplayer}
    return render(request, 'HockeyTracker/hockeytracker_details.html', content)
```
### Edit and Delete functions
```
def hockeytracker_edit(request, pk):
    hockeyplayer = get_object_or_404(HockeyPlayer, pk=pk)
    form = HockeyPlayerForm(data=request.POST or None, instance=hockeyplayer)
    if request.method == 'POST':
        if form.is_valid():
            form.save()
            return redirect('../../read')
        else:
            print(form.errors)
    content = {'form': form, 'hockeyplayer': hockeyplayer}
    return render(request, 'HockeyTracker/hockeytracker_edit.html', content)

def hockeytracker_delete(request, pk):
    hockeyplayer = get_object_or_404(HockeyPlayer, pk=pk)
    if request.method == 'POST':
        hockeyplayer.delete()
        return redirect('../../read')
    content = {'hockeyplayer': hockeyplayer}
    return render(request, 'HockeyTracker/hockeytracker_delete.html', content)
```
<img width="1425" alt="Screen Shot 2023-01-19 at 9 54 56 AM" src="https://user-images.githubusercontent.com/36861079/213522759-152f9f01-7efb-4de4-8340-ef8bdb38391b.png">

<br>

# Accessing the NHL.com API:

<br>

## The view function to request a list of all NHL teams from the API:
```
def hockeytracker_nhl(request):
    url = "https://statsapi.web.nhl.com/api/v1/teams"
    response = requests.request("GET", url)

    api_response = response.json()
    teamslist = api_response["teams"]

    content = {
        "teamslist": teamslist
    }
    return render(request, 'HockeyTracker/hockeytracker_nhl.html', content)
```
## The Django template code to render the teams results in the form of an HTML table:
```
{% extends "hockeytracker_base.html" %}

{% block content %}
<h1>NHL.com API</h1>
<h2>Current NHL Teams</h2>
<hr>
<h2>Click on any row to view stats and recent game content from your favorite NHL team!</h2>
<table id="sorttable">
    <tr>
        <th>Team</th>
        <th>Conference</th>
        <th>Division</th>
    </tr>
    {% for i in teamslist%}
    <tr onclick="location.href='{% url 'hockeytracker_roster' i.id i.name %}';">
        <td>{{ i.name }}</td>
        <td>{{ i.conference.name }}</td>
        <td>{{ i.division.name }}</td>
    </tr>
    {% endfor %}
</table>
{% endblock %}
```
<img width="1424" alt="Screen Shot 2023-01-19 at 9 57 51 AM" src="https://user-images.githubusercontent.com/36861079/213523337-b5ee3f18-4fa5-41dd-9daf-1d842dda7fd8.png">

<br>

## Javascript function to sort tables alphabetically when displayed in the application:
```
function sortTable() {
  var table, rows, switching, i, x, y, shouldSwitch;
  table = document.getElementById("sorttable");
  switching = true;
  while (switching) {
    switching = false;
    rows = table.rows;
    for (i = 1; i < (rows.length - 1); i++) {
      shouldSwitch = false;
      x = rows[i].getElementsByTagName("TD")[0];
      y = rows[i + 1].getElementsByTagName("TD")[0];
      if (x.innerHTML.toLowerCase() > y.innerHTML.toLowerCase()) {
        shouldSwitch = true;
        break;
      }
    }
    if (shouldSwitch) {
      rows[i].parentNode.insertBefore(rows[i + 1], rows[i]);
      switching = true;
    }
  }
}
sortTable();
```

<br>

## The view function API request for the roster, most recent game date, score, and primary key for the team selected by the user:
```
ef hockeytracker_roster(request, x, teamname):
    url = "https://statsapi.web.nhl.com/api/v1/teams/" + str(x) + "/roster"
    response = requests.request("GET", url)

    api_response = response.json()
    roster = api_response["roster"]

    # Add in an additional API call to find previous game for the team and return the boxscore API results:

    url2 = "https://statsapi.web.nhl.com/api/v1/teams/" + str(x) + "?expand=team.schedule.previous"
    response2 = requests.request("GET", url2)

    api_response2 = response2.json()
    previousgamepk = api_response2["teams"][0]["previousGameSchedule"]["dates"][0]["games"][0]["gamePk"]
    previousgamedate = api_response2["teams"][0]["previousGameSchedule"]["dates"][0]["date"]

    url3 = "https://statsapi.web.nhl.com/api/v1/game/" + str(previousgamepk) + "/boxscore"
    response3 = requests.request("GET", url3)
    api_response3 = response3.json()
    previousgamebox = api_response3["teams"]

    content = {
        "roster": roster,
        "teamname": teamname,
        "previousgamebox": previousgamebox,
        "previousgamepk": previousgamepk,
        "previousgamedate": previousgamedate
    }
    return render(request, 'HockeyTracker/hockeytracker_roster.html', content)
```
## The Django template code to render the selected team's most recent game score, link to video highlights, and roster:
```
{% extends "hockeytracker_base.html" %}

{% block content %}
<h1>{{ teamname }}</h1>
<h2>Most Recent Game: {{ previousgamedate }}</h2>
<h3>{{ previousgamebox.away.team.name }} vs {{ previousgamebox.home.team.name }}</h3>
<table class="boxtable">
    <tr class="boxrow">
        <th class="boxheader"></th>
        <th class="boxheader">{{ previousgamebox.away.team.name }}</th>
        <th class="boxheader">{{ previousgamebox.home.team.name }}</th>
    </tr>
    <tr class="boxrow">
        <td class="boxlabel">Score</td>
        <td class="boxgoals"><div class="divgoals">{{ previousgamebox.away.teamStats.teamSkaterStats.goals }}</div></td>
        <td class="boxgoals"><div class="divgoals">{{ previousgamebox.home.teamStats.teamSkaterStats.goals }}</div></td>
    </tr>
    <tr class="boxrow">
        <td class="boxlabel">Shots</td>
        <td class="boxshothit">{{ previousgamebox.away.teamStats.teamSkaterStats.shots }}</td>
        <td class="boxshothit">{{ previousgamebox.home.teamStats.teamSkaterStats.shots }}</td>
    </tr>
    <tr class="boxrow">
        <td class="boxlabel">Hits</td>
        <td class="boxshothit">{{ previousgamebox.away.teamStats.teamSkaterStats.hits }}</td>
        <td class="boxshothit">{{ previousgamebox.home.teamStats.teamSkaterStats.hits }}</td>
    </tr>
</table>
<button onclick="location.href='{% url 'hockeytracker_highlights' previousgamepk %}';">Click HERE for Video Highlights!</button>
<hr>
<h2>Current Roster</h2>
<h3>Click on any row to view the player's stats and reveal the SAVE option!</h3>
<table id="sorttable">
    <tr>
        <th>Name</th>
        <th>Position</th>
        <th>Jersey Number</th>
    </tr>
    {% for i in roster%}
    <tr onclick="location.href='{% url 'hockeytracker_playerstats' i.person.id i.person.fullName %}';">
        <td>{{ i.person.fullName }}</td>
        <td>{{ i.position.name }}</td>
        <td>{{ i.jerseyNumber }}</td>
    </tr>
    {% endfor %}
</table>
{% endblock %}
```
<img width="1424" alt="Screen Shot 2023-01-19 at 10 02 54 AM" src="https://user-images.githubusercontent.com/36861079/213525757-c89fac59-d673-4f7d-8b9e-1ce511382c5a.png">

<img width="1422" alt="Screen Shot 2023-01-19 at 10 03 21 AM" src="https://user-images.githubusercontent.com/36861079/213525816-2d1a3b29-862f-48c2-8c3f-3e03ea463d5d.png">

<br>

## The view function API request for the selected player's stats, with the code to save the stats to the Hocky Player database model:
```
def hockeytracker_saveplayerstats(request, x, playername):
    url = "https://statsapi.web.nhl.com/api/v1/people/" + str(x) + "/stats?stats=statsSingleSeason"
    response = requests.request("GET", url)

    api_response = response.json()
    savestatdict = api_response["stats"][0]["splits"][0]['stat']
    hockeyplayer = HockeyPlayer(
        name=playername,
        timeonice=savestatdict['timeOnIce'],
        points=savestatdict['points'],
        goals=savestatdict['goals'],
        assists=savestatdict['assists'],
        hits=savestatdict['hits'],
        faceoffpct=savestatdict['faceOffPct'],
        plusminus=savestatdict['plusMinus']
    )
    hockeyplayer.save()
    return redirect('../../../read')
```

<br>

## The view function API request for the URL associated with a team's most recent game video highlights:
```
def hockeytracker_highlights(request, previousgamepk):
    url = "https://statsapi.web.nhl.com/api/v1/game/" + str(previousgamepk) + "/content"
    response = requests.request("GET", url)

    api_response = response.json()
    videourl = api_response["media"]["epg"][2]["items"][0]["playbacks"][3]["url"]

    content = {
        "videourl": videourl
    }

    return render(request, 'HockeyTracker/hockeytracker_highlights.html', content)
```
<img width="1419" alt="Screen Shot 2023-01-19 at 10 16 54 AM" src="https://user-images.githubusercontent.com/36861079/213527250-82b6f716-d145-462c-99b8-237c3406499b.png">
