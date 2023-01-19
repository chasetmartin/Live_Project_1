# Live_Project_1
Repository for code snippets and a description of work I accomplished during my 1st Tech Academy Live Project

## Creating a Django Model and Model Manager:
'''
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
'''
