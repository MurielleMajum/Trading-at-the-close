# Trading-at-the-close

L'objectif ici est de développer un modèle capable de prédire les mouvements des prix de clôture pour des centaines d'actions du Nasdaq en utilisant les données du carnet d'ordres et de l'enchère de clôture des actions.

Nous avons ci-dessous une brève explication des variables:
# Description des Variables

- **stock_id** : Identifiant unique pour chaque action.

- **date_id** : Identifiant unique pour chaque jour de trading.

- **seconds_in_bucket** : Le nombre de secondes écoulées depuis le début d'un intervalle de temps donné (par exemple, une minute ou une heure). Cela peut être utilisé pour situer les transactions ou les ordres dans le temps au cours de la journée de trading.

- **imbalance_size** : La taille de l'imbalance, c'est-à-dire la différence entre les volumes d'ordres d'achat et de vente à un moment donné. Une imbalance peut indiquer une pression acheteuse ou vendeuse sur le marché.

- **imbalance_buy_sell_flag** : Indicateur qui montre si l'imbalance est du côté des acheteurs (valeurs positives) ou des vendeurs (valeurs négatives).

- **reference_price** : Le prix de référence utilisé pour certains calculs ou comparaisons. Cela peut être un prix moyen, le dernier prix négocié, ou un prix de référence défini par l'échange.

- **matched_size** : Le volume total des transactions effectuées à un prix donné. Cela représente la quantité d'actions échangées.

- **far_price** : Le prix le plus élevé dans le carnet d'ordres, souvent appelé le prix de l'offre la plus éloignée.

- **near_price** : Le prix le plus bas dans le carnet d'ordres, souvent appelé le prix de la demande la plus proche.

- **bid_price** : Le prix d'achat le plus élevé actuellement disponible dans le carnet d'ordres. Les traders souhaitant acheter au prix de l'offre achèteront à ce prix.

- **bid_size** : La quantité d'actions disponibles à l'achat au prix de l'offre.

- **ask_price** : Le prix de vente le plus bas actuellement disponible dans le carnet d'ordres. Les traders souhaitant vendre au prix de la demande vendront à ce prix.

- **ask_size** : La quantité d'actions disponibles à la vente au prix de la demande.

- **wap** (Weighted Average Price) : Le prix moyen pondéré des transactions:

$$
\text{wap} = \frac{\text{BidPrice} \times \text{AskSize} + \text{AskPrice} \times \text{BidSize}}{\text{BidSize} + \text{AskSize}}
$$

- **target** : La variable cible que vous souhaitez prédire. Dans ce contexte, il s'agit ici de la variation ou du mouvement du prix de clôture de l'action:


$$
\text{Target} = (\frac{StockWAP_{t+60}}{StockWAP_{t}} - \frac{IndexWAP_{t+60}}{IndexWAP_{t}}) \times 1000
$$

- **time_id** : Un identifiant temporel qui est utilisé pour regrouper les données par tranche de temps spécifique.

- **row_id** : Un identifiant unique pour chaque ligne dans le dataset. Cela peut être utilisé pour référencer ou retrouver une ligne spécifique.



La métrique à utiliser ici sera formule pour l'erreur absolue moyenne (MAE) qui est donnée par :

$$
\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |y_i - x_i|
$$

où :
- \( n \) est le nombre de points de données,
- \( y_i \) est la valeur observée,
- \( x_i \) est la valeur prédite.

# Analyse descriptive

## Gestion des valeurs manquantes
Nos données ayant environ 6.5% de valeurs manquantes, nous alons utiliser des méthodes d'amputation pour inférer ces valeurs NA: une amputation par la moyenne/ médiane pour les variables imbalance_size, reference_price, matched_size, bid_price, ask_price, wap, target qui ont au plus 220/5237980 NA et une méthode d'interpolation pour les variables far_price et near_price qui ont environ 2894342/5237980 NA. _
Nous précisons que ce travail préliminaire est fait pour bien se familiariser avec les données, gagner en temps lors de l'apprentissage de nos modèles car les modèles utilisés ont déjà des options d'imputation de valeurs NA (et donc on pourrait négliger cette partie vu que les modèles le feront mais c'est mieux de le faire pour ganer en temps de compilation). 

## Correlation et Sélection de variables
Des plots, des boxplots et des histogrammes ont été réalisés et la matrice de confusion nous a montré les variables qui sont liées. Notamment reference_price, bid_price, wap et ask_price qui sont fortement correlées; idem pour date_id et time_id; imbalance_size et matched_size sont correlées; nous avons aussi near_price dont la correlation n'est pas négligeable avec reference_price, imbalance_buy_sell_flag, bid_price, ask_price et wap.\
Vu le grand nombre de nos données (plus de 5 millions), nous avons envisagé une sélection des variables qui influencent le plus dans la prédiction du prix de fermeture (target), ceci avec le le modèle Catboost.

# Modélisation
Nous avons utilisé deux modèles pour un début: un Catboost et un LBMboost. Ensuite nous avons optimisé les hyperparamètres avec une BayesSearch. Après obtension des paramètres optimaux (fonctions des plages choisies), nous avons fait des mélanges des deus modèles (stacking et voting classifier) pour essayer d'augmenter la performance.\
Le tableau ci dessous donne un petit récapitulatif des modèles testeés.


| Modèle              | MAE sur l'ensemble de validation |
|---------------------------------|----------------------------------|
| CatBoost                        | 6.243025914454114                          |
| LightGBM              | 6.296589689598023           |
| Stacking                    |                            |
|Voting Classifier|             |


Les meilleurs hyperparamètres obtenus:
| Modèle       | Hyperparamètre       | Valeur                         |
|--------------|----------------------|-------------------------------|
| CatBoost     | `iterations`         | 942                           |
|              | `learning_rate`      | 0.2541400145361314            |
|              | `depth`              | 9                             |
|              | `l2_leaf_reg`        | 4.0                           |
|              | `loss_function`      | 'MAE'                         |
|              | `verbose`            | 100                           |
| LightGBM     | `boosting_type`      | 'gbdt'                        |
|              | `objective`          | 'regression'                  |
|              | `num_leaves`         | 100                           |
|              | `learning_rate`      | 0.1                           |
|              | `max_depth`          | 9                             |
|              | `reg_lambda`         | 10.0                          |
|              | `verbose`            | 100                           |



















