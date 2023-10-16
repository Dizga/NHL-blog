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

La classe NHLDataDownloader dans le fichier load.py permet de télécharger les données d'une année donnée. Pour ce faire, on doit faire une instanciation de cette classe et on la donne comme paramétre l'année cible.

Il faut donc faire l'importation du fichier contenant cette classe.
```bash
    from src.data.load import NHLDataDownloader
  ``` 
Utiliser la fonction load_data() de cette classe pour télécharger les données. Ci dessous un exemple de code pour télécharger les données de la saison 2018.

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

Create a function to update the Plotly scatter plot based on the selected game
 game = season.regulars[game_idx]
    df = game.to_df()
    match_title = f'{game.home_team.name} VS {game.away_team.name}'

    fig = px.scatter(df, x="x", y="y", animation_frame="event idx", range_x=[-100,100], range_y=[-42.5,42.5])

    fig.update_traces(mode='markers',
                             marker_size=15,
                             marker_color="#111111")

    fig.add_layout_image(
      source="https://raw.githubusercontent.com/udem-ift6758/project-template/main/figures/nhl_rink.png",
      xref="x",
      yref="y",
      x=-100,
      y=42.5,
      sizex=200,
      sizey=85,
      sizing="stretch",
      opacity=0.9,
      layer="below"
    )
    fig.update_xaxes(showline=False, zeroline=False, showgrid=False, range=[-100, 100])
    fig.update_yaxes(
        showline=False,
        zeroline=False,
        showgrid=False,
        range=[-42.5, 42.5],
        scaleanchor = "x",
        scaleratio = 1,
      )

    fig.update_layout(
      title={
            'text': match_title,
            'y':0.9,
            'x':0.5,
            'xanchor': 'center',
            'yanchor': 'top'},
      autosize=False,
      template="plotly_white",)

    fig.layout.updatemenus[0].buttons[0].args[1]["frame"]["duration"] = 2000
    fig.layout.updatemenus[0].buttons[0].args[1]["frame"]["redraw"] = True

    for id, fr in enumerate(fig.frames):
      fr.layout.title = df.loc[id]['description']

    for step in fig.layout.sliders[0].steps:
        step["args"][1]["frame"]["redraw"] = True

    fig.show()


Create a dropdown widget to select the match
match_dropdown = widgets.Dropdown(options=range(len(season.regulars)), description="Select Match:")

Create an interactive output that displays the plot
interactive_output = widgets.interactive_output(update_plot, {'game_idx': match_dropdown})

Display the widgets
widgets.VBox([match_dropdown, interactive_output])
```
Cette figure représente des événements au cours d'un match de hockey. Les données sont affichées sous forme de nuage de points animé, où chaque point correspond à un événement particulier d'un match choisi. Les événements sont disposés en fonction de leurs coordonnées x et y, et l'animation permet de suivre l'évolution de ces événements au fil du temps en se basant sur l'indice de l'événement.
![Interactive hockey match](/assets/images/interactive.png)
Cette figure illustre le suivi des événements pour un match donné de manière interactive. 
{% include plotly_demo_3.html %}

## Nettoyer les données 
### Question 1
L'illustration ci-dessous présente un extrait du dataframe final, offrant ainsi un aperçu des données que nous avons obtenues.
![Dataframe](/assets/images/df.png)

### Question 2 
Afin d'améliorer l'analyse des matchs hockey, on peut ajouter des informations sur la force réelle (comme 5 contre 4, 5 contre 3) aux tirs et aux buts. On peut citer quelques approches possibles:

1. Suivi de l'état du match : Créer un système de suivi en temps réel pour enregistrer le nombre de joueurs sur la glace pour chaque équipe. Ces données seront associées à chaque tir ou but. Par exemple, en cas d'avantage numérique, l'état du match passera à 5 contre 4, une telle information sera liée à tous les événements durant cet avantage.

2. Utilisation des pénalités : Les données sur les pénalités font généralement partie des données de match de hockey. En utilisant les heures de début et de fin des pénalités, on pourrait déterminer l'état du match pendant ces périodes. Lorsqu'une pénalité est en cours, l'état du match serait ajusté pour refléter le changement de force des équipes (par exemple, 5 contre 4 pour l'équipe en supériorité numérique). Cela dépend de la précision des données sur les pénalités.

3. Terminologie standardisée : Créer une terminologie standard pour différents états du match, comme "5v5," "5v4," "4v5,". Assurer une utilisation cohérente de cette terminologie lors de la collecte de données.


### Question 3
Nous envisageons d'explorer trois aspects de l'évaluation des tirs : la classification des tirs en contre-attaque, l'analyse des tirs de rebond et l'identification des tirs à longue distance. 

1. Classification des tirs en contre-attaque : En analysant la séquence d'événements, il serait possible de classifier les tirs en tant que tirs en contre-attaque. Cette analyse se base sur la rapidité avec laquelle l'équipe attaquante récupère la rondelle après un revirement, créant ainsi une opportunité de tir rapide. Cela permet d'évaluer l'efficacité de l'équipe dans ces situations clés.


2.  des tirs de rebond : Ces tirs résultent souvent d'un tir initial repoussé par le gardien de but ou la structure du but. Ils créent des opportunités de but supplémentaires et permettent d'évaluer la performance du gardien à les bloquer.

3. Identification des tirs à longue distance : Cela évalue la précision et l'efficacité des tirs de la périphérie de la zone offensive par rapport aux tirs plus proches du but, offrant des indications sur le style de jeu de l'équipe.

## Visualisations simples

###  Question 1
Le Tip-In est le type de tir le plus dangereux en termes de conversion en buts parmi les types de tirs analysés. En effet, en 2018, 17.16% des tirs de type Tip-In sont des buts. En revanche, le Wrist Shot est le type de tir le plus dangereux en termes de nombre total de buts. On remarque également que Wrist Shot est également le type de tir le plus courant.

Le choix du graphique à barres est expliqué pour plusieurs raisons. Tout d'abord, il permet une comparaison directe des types de tirs en superposant les barres de buts sur celles des tirs, mettant ainsi en lumière la proportion de buts par rapport aux tirs. Cette visualisation facilite l'identification des tendances et la mise en évidence des types de tirs les plus courants et les plus dangereux.
De plus, l'utilisation d'une échelle logarithmique pour l'axe des ordonnées permet de visualiser efficacement des valeurs très différentes entre les types de tirs.

En résumé, le graphique à barres empilées offre une perspective claire et concise pour analyser les données sur les tirs et les buts.

![Shots types](/assets/images/2018.png)

### Question 2
Plus la rondelle est proche au filet (la distance est minimale), plus la probabilité d'avoir un but augmente. Cette relation entre la distance à laquelle un tir a été effectué et la chance qu'il s'agisse d'un but n'a pas eu de changement remarquables d'une saison à une autre.  

![](/assets/images/2018_dist.png)
![](/assets/images/2019_dist.png)
![](/assets/images/2020_dist.png)


### Question 3 
![](/assets/images/5.3.png)
Cette figure montre le pourcentage de buts en fonction de la distance par rapport au filet et de la catégorie de types de tirs pendant la saison 2018. Il est observé que le type de tir "Deflected" est le plus dangereux. En effet, pour toutes les plages de distances, les chances de marquer un but avec ce type de tir sont relativement élevées.
Les deux types de tir slap shot et snap shot s'avèrent etre dangereux aussi à des intervalles de distance faibles. 


## Visualisations avancées

### Question 1 
Ces figures illustrent les 4 graphiques au zone offensive de la saison 2016/2017 jusqu'à la saison 2020/2021.
{% include 20162017.html %}

{% include 20172018.html %}

{% include 20182019.html %}

{% include 20192020.html %}

{% include 20202021.html %}

### Question 2 
Dans les quatres saisons, on remarque que le cœur de territoire de l’adversaire est souvent en bleu, c’est à dire que aucun tir est presque fait à partir de cette zone. En fait, Les tirs sont souvent fait des côtés de terrains. Cette observation semble logique puisque en réalité, la densité des joueurs en cette partie de terrain est élevé.


### Question 3
La carte des tirs de l'Avalanche du Colorado au cours de la saison 2016-17 révèle une performance offensive limitée. La plupart des tirs ont été effectués depuis des positions éloignées du filet, ce qui indique des difficultés à percer les défenses adverses. Cette performance s'est reflétée dans un mauvais classement. En fait, l’équipe a terminé la saison régulière à la 30e et dernière place de la ligue, ce qui signifie qu’elle n’a pas réussi à se qualifier pour les séries éliminatoires.

À l’inverse, la carte des tirs pour la saison 2020-21 indique un style de jeu plus agressif et offensif de la part de l’Avalanche du Colorado. L'équipe a réussi à réaliser des tirs depuis de nombreuses positions proches du filet, démontrant ainsi sa capacité à contrôler le jeu et à pénétrer les défenses adverses. Ces performances ont permis à l’équipe de s’assurer la première place du championnat. 

### Question 4 

Les styles de jeu de ces deux équipes sont clairement différents. La distribution de tirs du Lightning de Tampa Bay souligne son approche offensive agressive. Ils ont excellé dans l’exercice de la pression sur l’équipe adverse, en maintenant le contrôle territorial et en faisant des tris proches du filet. En revanche, la carte des tirs des Sabres de Buffalo suggère un style de jeu plus passif et moins efficace. La majorité de leurs tirs sont lancés depuis les côtés de la patinoire, souvent loin du filet adverse. Cette différence notable dans l’agressivité offensive et la domination est un facteur clé contribuant aux vastes disparités dans leurs classements respectifs.

Cependant, il est crucial de reconnaître que si les cartes de tir ne fournissent qu'une vue partielle de la performance globale d'une équipe. Pour comprendre globalement les succès ou les difficultés d'une équipe de la LNH, divers facteurs supplémentaires doivent être pris en compte :

1. Compétence et profondeur des joueurs : Le talent et la profondeur de l'effectif d'une équipe sont essentiels au succès.

2. Force de l'adversaire : La qualité des équipes auxquelles ils font face joue un rôle important.

3. Blessures des joueurs : les blessures des joueurs clés peuvent affecter considérablement les performances. Surveiller la santé des joueurs vedettes est essentiel.
