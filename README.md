# Live Project - Django App - NHL Hockey Statistic Tracker
Repository for code snippets and a description of work I accomplished during my 1st Tech Academy Live Project. I worked on bulding a single Django application within a larger group project, and the focus of my application was creating a place for user's to store data about local hockey player, or view stats and highlights for their favorite NHL teams and players using the NHL.com API.

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

## Writing view functions to create, read, edit, and delete instances of my Hockey Player Model:
### Story #2: Create model and render create entry page
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
### Story #3: Display all items from database
```
def hockeytracker_read(request):
    hockeyplayer = HockeyPlayer.HockeyPlayers.all()
    content = {'hockeyplayer': hockeyplayer}
    return render(request, 'HockeyTracker/hockeytracker_read.html', content)
```
<img width="1421" alt="Screen Shot 2023-01-19 at 9 50 14 AM" src="https://user-images.githubusercontent.com/36861079/213521751-cee2d414-ddf3-4a43-8852-f629bc9010f8.png">

### Story #4: Details Page for selected player
```
def hockeytracker_details(request, pk):
    hockeyplayer = get_object_or_404(HockeyPlayer, pk=pk)
    content = {'hockeyplayer': hockeyplayer}
    return render(request, 'HockeyTracker/hockeytracker_details.html', content)
```
### Story #5: Edit and Delete functions
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
