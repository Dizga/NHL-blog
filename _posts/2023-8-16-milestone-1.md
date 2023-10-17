---
layout: post
title: Milestone 1
---

## Acquisition de données 

Afin de télécharger les données de match NHL, nous avons suivi les étapes comme décrites ci-dessous :

Pour commencer, il faut installer les bibliothèques requises en exécutant 

  ```bash
    pip install -r requirement
  ``` 

Pour télécharger les données d'une année spécifique, il suffit d'utiliser la classe NHLDataDownloader dans le fichier load.py. Pour ce faire, on doit faire une instanciation de cette classe et on la donne comme paramétre l'année cible.

Il faut donc faire l'importation du fichier contenant cette classe.
```bash
    from src.data.load import NHLDataDownloader
  ``` 
Utiliser la fonction load_data() de cette classe pour télécharger les données. 

Ci-dessous un exemple de code pour télécharger les données de la saison 2018.

```python
 # Instancier un objet de la classe NHLDataDownloader en spécifiant l'année cible 
      season_year = 2017
      nhl_downloader = NHLDataDownloader(season_year)
      # télécharger les données
      season_data = nhl_downloader.load_data()   
```    

     
Si les données sont déjà présentées dans un fichier, elles sont chargées. Sinon, elles sont téléchargées et enregistrées dans un fichier.

Une fois le téléchargement terminé, les données seront enregistrées dans un fichier au format season_fullname.pkl, où season_fullname est le nom complet de la saison (par exemple, 20172018.pkl). Vous pouvez maintenant utiliser ces données selon vos besoins


## Outil de débogage interactif 

L'exemple de code ci-dessous implémente la fonctionnalité d'un outil de débogage interactif qui permet de parcourir tous les événements pour chaque match d'une saison spécifique.

```python
rink_image = Image.open("figures/nhl_rink.png")

pp = pprint.PrettyPrinter(indent=4)

data = season_data

# Dropdown to select 'regulars' or 'playoffs'
type_dropdown = widgets.Dropdown(options=['regulars', 'playoffs'],
                                 description='Type:')

# Slider to switch between different games
game_slider = widgets.IntSlider(value=0,
                                min=0,
                                max= len(data['regulars']) - 1,
                                description='Game Index:')

# Slider to switch between different plays
play_slider = widgets.IntSlider(value=0,
                                min=0,
                                max=len(data[
                                    'regulars'][0][
                                        'liveData'][
                                            'plays'][
                                                'allPlays']) - 1,
                                description='Play Index:')

# Output widget to display the plot
plot_output = widgets.Output()

def update_game_slider_range(*args):
    """Update the game slider range based on the type selected."""
    test = {"test":"regulars"}
    if type_dropdown.value == 'regulars':
        game_slider.max = len(data['regulars']) - 1
    else:
        game_slider.max = len(data['playoffs']) - 1
type_dropdown.observe(update_game_slider_range, 'value')

def update_play_slider_range(*args):
    """Update the play slider range based on game_slider's value."""
    if type_dropdown.value == 'regulars':
        coords_length = len(data[
            'regulars'][
                game_slider.value][
                    'liveData'][
                        'plays'][
                            'allPlays']) - 1
    else:
        key = list(data['playoffs'].keys())[game_slider.value]
        coords_length = len(data[
            'playoffs'][
                key][
                    'liveData'][
                        'plays'][
                            'allPlays']) - 1
    
    play_slider.max = coords_length - 1
game_slider.observe(update_play_slider_range, 'value')

def plot_coordinates(change):
    """Plot the coordinates based on the selected options."""
    with plot_output:
        plot_output.clear_output(wait=True)

        if type_dropdown.value == 'regulars':
            game = data['regulars'][game_slider.value]

        else:
            key = list(data['playoffs'].keys())[game_slider.value]
            game = data['playoffs'][key]

        play = game[
            'liveData'][
                'plays'][
                    'allPlays'][
                        play_slider.value]
        coords = play['coordinates']
        teams = game['liveData']['linescore']['teams']

        home = teams["home"]["team"]["triCode"]
        home_score = teams["home"]["goals"]
        away = teams["away"]["team"]["triCode"]
        away_score = teams["away"]["goals"]

        print(f'{home} {home_score} goals vs {away} {away_score} goals')
        pp.pprint(play)

        if len(coords) == 0:
          coords = {'x':None, 'y':None}

        fig = go.Figure()

        fig.add_trace(go.Scatter(x= [coords['x']], y=[coords['y']],
                                mode='markers',
                                marker_size=15,
                                marker_color="#111111"))

        # Add images
        fig.add_layout_image(
                dict(
                    source=rink_image,
                    xref="x",
                    yref="y",
                    x=-100,
                    y=42.5,
                    sizex=200,
                    sizey=85,
                    sizing="stretch",
                    opacity=0.9,
                    layer="below")
        )

        fig.update_xaxes(
            showline=False,
            zeroline=False,
            showgrid=False,
            range=[-100, 100])
        fig.update_yaxes(
            showline=False,
            zeroline=False,
            showgrid=False,
            range=[-42.5, 42.5],
            scaleanchor = "x",
            scaleratio = 1,
          )

        fig.update_layout(
            autosize=False,
            template="plotly_white")

        fig.show()

# Watch for changes
type_dropdown.observe(plot_coordinates, 'value')
game_slider.observe(plot_coordinates, 'value')
play_slider.observe(plot_coordinates, 'value')

# Initial plot
plot_coordinates(None)

# Display widgets
display(type_dropdown, game_slider, play_slider, plot_output)
```
Cette figure représente des événements qui se déroulent pendant un match de hockey. Les données sont affichées sous forme de nuage de points animé, où chaque point correspond à un événement particulier d'un match choisi. Les événements sont disposés en fonction de leurs coordonnées x et y, et l'animation permet de suivre l'évolution de ces événements au fil du temps en se basant sur l'indice de l'événement.

![Interactive hockey match](/assets/images/debug_tool.png)

Cette figure illustre le suivi des événements pour un match donné de manière interactive. 

{% include plotly_demo_3.html %}

## Nettoyer les données 
### Question 1
L'illustration ci-dessous présente un extrait du dataframe final, offrant ainsi un aperçu des données que nous avons obtenues.
![Dataframe](/assets/images/df.png)

### Question 2 
Dans le but d'améliorer l'analyse des matchs hockey, il est envisageable d'inclure des informations sur la force réelle (telles que 5 contre 4, 5 contre 3) aux tirs et aux buts. On peut citer quelques approches possibles:

1. Suivi de l'état du match : Mettre en place un système de suivi en temps réel pour enregistrer le nombre de joueurs sur la glace pour chaque équipe. Ces données seront associées à chaque tir ou but. Dans le cas d'un avantage numérique, par exemple, le score du match passera à 5 contre 4, et toutes les informations relatives à cet avantage seront connectées à tous les événements qui se déroulent durant cette période.

2. Utilisation des pénalités : Généralement, les données relatives aux pénalités sont incluses dans les données de match de hockey.  En utilisant les temps de début et de fin des pénalités, il serait possible de déterminer l'état du match au cours de ces périodes. Lorsqu'une pénalité est en cours, l'état du match serait ajusté pour refléter le changement de force des équipes (par exemple, 5 contre 4 pour l'équipe en supériorité numérique). Cela dépend de la précision des données sur les pénalités.

3. Terminologie standardisée : Créer une terminologie standard pour différents états du match, comme "5v5," "5v4," "4v5,". Assurer une utilisation cohérente de cette terminologie lors de la collecte de données.


### Question 3
Nous envisageons d'explorer trois aspects de l'évaluation des tirs : la classification des tirs en contre-attaque, l'analyse des tirs de rebond et l'identification des tirs à longue distance. 

1. Classification des tirs en contre-attaque : En étudiant la séquence d'événements, il serait envisageable de catégoriser les tirs comme des tirs en contre-attaque. Cette analyse est basée sur la rapidité avec laquelle l'équipe attaquante récupère la rondelle après un revirement, créant ainsi une opportunité de tir rapide. Cela permet d'évaluer l'efficacité de l'équipe dans ces situations critiques.


2.  des tirs de rebond : Ces tirs sont souvent le résultat d'un tir initial repoussé par le gardien de but ou la structure du but. Ils créent des opportunités de but supplémentaires et permettent d'évaluer la performance du gardien à les bloquer.

3. Identification des tirs à longue distance : Cela évalue la précision et l'efficacité des tirs de la périphérie de la zone offensive par rapport aux tirs plus proches du but, offrant des indications sur le style de jeu de l'équipe.

## Visualisations simples

###  Question 1
Le Tip-In est le type de tir le plus dangereux en matière de conversion en buts parmi les types de tirs analysés. En effet, en 2018, 17.16% des tirs du type Tip-In sont des buts. En revanche, le Wrist Shot est le type de tir le plus dangereux en termes de nombre total de buts. On remarque également que Wrist Shot est également le type de tir le plus courant.

Le choix du graphique à barres est expliqué par un certain nombre de raisons. Tout d'abord, il permet une comparaison directe des types de tirs en superposant les barres de buts sur celles des tirs, il met  ainsi en evidence la proportion de buts par rapport aux tirs. Une telle visualisation facilite l'identification des tendances et la mise en oeuvre des types de tirs les plus courants et les plus dangereux.
De plus, l'utilisation d'une échelle logarithmique pour l'axe des ordonnées permet de visualiser efficacement des valeurs très différentes entre les types de tirs.

En résumé, le graphique à barres offre une perspective claire et concise pour analyser les données sur les tirs et les buts.

<!-- ![Shots types](/assets/images/2018.png) -->
{% include type-shot-2018.html %}

### Question 2
Plus la rondelle est proche au filet (la distance est minimale), plus la probabilité d'avoir un but augmente. Cette relation entre la distance à laquelle un tir a été effectué et la chance qu'il s'agisse d'un but n'a pas eu de changement remarquables d'une saison à une autre.  


![](/assets/images/shot-distance-2018.png)
![](/assets/images/shot-distance-2019.png)
![](/assets/images/shot-distance-2020.png)


### Question 3 
<!-- ![](/assets/images/5.3.png) -->
{% include type-shot-distance-2020.html %}
Cette figure montre le pourcentage de buts en fonction de la distance par rapport au filet et de la catégorie de types de tirs pendant la saison 2018. Il est observé que le type de tir "Deflected" est le plus dangereux. En effet, pour toutes les plages de distances, les chances de marquer un but avec ce type de tir sont relativement élevées.
Les deux types de tir slap shot et snap shot s'avèrent être  dangereux aussi à des intervalles de distance faibles. 


## Visualisations avancées

### Question 1 
Ces figures illustrent les  graphiques au zone offensive de la saison 2016/2017 jusqu'à la saison 2020/2021.
{% include 20162017.html %}

{% include 20172018.html %}

{% include 20182019.html %}

{% include 20192020.html %}

{% include 20202021.html %}

### Question 2 
Dans les quatre saisons, on remarque que le cœur de territoire de l’adversaire est souvent en bleu, c'est-à-dire qu'aucun tir est presque fait à partir de cette zone. En fait, Les tirs sont souvent fait des côtés de terrains. Cette observation semble logique puisque en réalité, la densité des joueurs en cette partie de terrain est élevé.


### Question 3
On peut observer une performance offensive limitée sur la carte des tirs de l'Avalanche du Colorado pour la saison 2016-17. La majorité des tirs ont été réalisés depuis des positions éloignées du filet, ce qui suggère des difficultés à percer les défenses adverses. Cette performance s'est reflétée dans un mauvais classement. En fait, l’équipe a terminé la saison régulière à la 30e et dernière place de la ligue, ce qui signifie qu’elle n’a pas réussi à se qualifier pour les séries éliminatoires.

Inversement, la carte des tirs pour la saison 2020-21 indique un style de jeu plus agressif et offensif de la part de l’Avalanche du Colorado. L'équipe a réussi à réaliser des tirs depuis de nombreuses positions proches du filet, démontrant ainsi sa capacité à contrôler le jeu et à pénétrer les défenses adverses. Ces performances ont permis à l’équipe de s’assurer la première place du championnat. 

### Question 4 

Les styles de jeu de ces deux équipes sont clairement différents. La distribution de tirs du Lightning de Tampa Bay souligne son approche offensive agressive. Ils ont excellé dans l’exercice de la pression sur l’équipe adverse, en maintenant le contrôle territorial et en faisant des tris proches du filet. En revanche, la carte des tirs du Sabres de Buffalo suggère un style de jeu plus passif et moins efficace. 

La plupart de leurs tirs sont effectués depuis les côtés de la patinoire, souvent en dehors du filet adverse. La présence de cette nette différence dans l'agressivité offensive et la domination joue un rôle majeur dans les écarts considérables de leurs classements respectifs.

Cependant, il est essentiel de reconnaître que si les cartes de tir ne donnent qu'une vision partielle de la performance globale d'une équipe. Pour une compréhension globale des succès ou des difficultés d'une équipe de la LNH, il est essentiel de prendre en compte d'autres facteurs supplémentaires : :

1. Compétence et profondeur des joueurs : Le talent et la profondeur de l'effectif d'une équipe sont essentiels au succès.

2. Force de l'adversaire : La qualité des équipes auxquelles ils font face joue un rôle important.

3. Blessures des joueurs : les blessures des joueurs clés peuvent affecter considérablement les performances. Surveiller la santé des joueurs vedettes est essentiel.
