---
layout: post
title: Milestone 1
---

## Question 1

...content...

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
Cette figure représente des événements au cours d'un match de hockey. Les données sont affichées sous forme de nuage de points animé, où chaque point correspond à un événement particulier d'un match choisi. Les événements sont disposés en fonction de leurs coordonnées x et y, et l'animation permet de suivre l'évolution de ces événements au fil du temps en se basant sur l'indice de l'événement.
![Interactive hockey match](/assets/images/debug_tool.png)
Cette figure illustre le suivi des événements pour un match donné de manière interactive. 
{% include plotly_demo_3.html %}

## Nettoyer les données 
### Question 1
L'illustration ci-dessous présente un extrait du dataframe final, offrant ainsi un aperçu des données que nous avons obtenues.
![Dataframe](/assets/images/df.png)

### Question 2 

1. Suivi de l'état du match : Créer un système de suivi en temps réel pour enregistrer le nombre de joueurs sur la glace pour chaque équipe. Ces données seraient associées à chaque tir ou but. Par exemple, en cas d'avantage numérique, l'état du match passerait à 5 contre 4, et cette information serait liée à tous les événements pendant cet avantage.

2. Utilisation des pénalités : Les données sur les pénalités font généralement partie des données de match de hockey. En utilisant les heures de début et de fin des pénalités, on pourrait déterminer l'état du match pendant ces périodes. Lorsqu'une pénalité est en cours, l'état du match serait ajusté pour refléter le changement de force des équipes (par exemple, 5 contre 4 pour l'équipe en supériorité numérique). Cela dépend de la précision des données sur les pénalités.

3. Terminologie standardisée : Créer une terminologie standard pour différents états du match, comme "5v5," "5v4," "4v5," serait essentiel. Assurer une utilisation cohérente de cette terminologie lors de la collecte de données.


### Question 3
Nous envisageons d'explorer trois aspects de l'évaluation des tirs : la classification des tirs en contre-attaque, l'analyse des tirs de rebond et l'identification des tirs à longue distance. 

1. Classification des tirs en contre-attaque : En analysant la séquence d'événements, il serait possible de classifier les tirs en tant que tirs en contre-attaque. Cette analyse se base sur la rapidité avec laquelle l'équipe attaquante récupère la rondelle après un revirement, créant ainsi une opportunité de tir rapide. Cela permet d'évaluer l'efficacité de l'équipe dans ces situations clés.


2.  des tirs de rebond : Ces tirs résultent souvent d'un tir initial repoussé par le gardien de but ou la structure du but. Ils créent des opportunités de but supplémentaires et permettent d'évaluer la performance du gardien à les bloquer.

3. Identification des tirs à longue distance : Cela évalue la précision et l'efficacité des tirs de la périphérie de la zone offensive par rapport aux tirs plus proches du but, offrant des indications sur le style de jeu de l'équipe.

## Visualisations simples

###  Question 1
Le Tip-In est le type de tir le plus dangereux en termes de conversion en buts parmi les types de tirs analysés. En effet, en 2018, 17.16% des tirs de type Tip-In sont des buts. En revanche, le Wrist Shot est le type de tir le plus dangereux en termes de nombre total de buts. On remarque également que Wrist Shot est également le type de tir le plus courant.
Le choix du graphique à barres est expliqué pour plusieurs raisons. Il permet une comparaison directe des types de tirs en superposant les barres de buts sur celles des tirs, mettant ainsi en lumière la proportion de buts par rapport aux tirs. Cette visualisation claire facilite l'identification des tendances et la mise en évidence des types de tirs les plus courants et les plus dangereux.
De plus, l'utilisation d'une échelle logarithmique pour l'axe des ordonnées permet de visualiser efficacement des valeurs très différentes entre les types de tirs. En résumé, le graphique à barres empilées offre une perspective claire et concise pour analyser les données sur les tirs et les buts.

![Shots types](/assets/images/2018.png)

### Question 2