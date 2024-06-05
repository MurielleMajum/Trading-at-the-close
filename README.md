# Trading-at-the-close

L'objectif ici est de développer un modèle capable de prédire les mouvements des prix de clôture pour des centaines d'actions du Nasdaq en utilisant les données du carnet d'ordres et de l'enchère de clôture des actions.\
La métrique à utiliser ici sera formule pour l'erreur absolue moyenne (MAE) qui est donnée par :

$$
\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |y_i - x_i|
$$

où :
- \( n \) est le nombre de points de données,
- \( y_i \) est la valeur observée,
- \( x_i \) est la valeur prédite.
