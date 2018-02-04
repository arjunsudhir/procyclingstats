# ProCyclingStats Points Prediction

## Objective
The goal of this project was to determine how accurately I could predict [ProCyclingStats](https://www.procyclingstats.com/) points with linear regression using features scraped from the race startlist, rider resume, and individual rankings pages of "the single most useful cycling database on the worldwide web" ([Rouleur, 2017](https://rouleur.cc/editorial/number-crunchers-pro-cycling-stats/)).

## Tools
- Requests for pulling content
- BeautifulSoup for saving and parsing HTML
- Numpy and Pandas for data manipulation
- Statsmodels and Scikit-learn for modeling
- Matplotlib and Seaborn for plotting

## Scraping & Parsing
I used the following races' startlists to generate a list of riders whose profile pages I would scrape for PCS points by season as well as a few other characteristics for my model.

**Startlists** - 85 total
- Since 2007: Giro d'Italia, Tour de France
- Since 2010: Vuelta a España
- Since 2013: Amstel Gold Race, Gent–Wevelgem, Il Lombardia, Liège–Bastogne–Liège, Milan–San Remo, Paris–Nice, Paris–Roubaix, Ronde van Vlaanderen, Strade Bianche, Tour of California, Tour de Suisse

The output was 1,730 unique riders which comprised 17,806 pages for about 10 seasons of data per rider. I also scraped the PCS individual rankings for each year since 2005 totaling 292 pages; however this ranking data has not been incorporated into my modeling yet.

**Features per Rider** - those in italics were parsed but not loaded into model
- rider name
- _team name_
- nationality
- date of birth (converted to age by year)
- _height_
- _weight_
- lifetime points by category: one day, general classification, time trial, sprint
- total race seasons
- _dates raced by year_
- _race names by year_
- _distance raced by year_
- _number of race days by year_
- _number of stage races by year_
- _UCI points won by year_
- PCS points won by year

## Modeling & Evaluation
The target variable (S0) for each rider was their most recently completed full racing season's PCS points (i.e. 2018 was excluded). This allowed the inclusion of retired riders alongside currently active ones, despite their careers spanning different years without overlap, with S1, S2, etc. as relative references to prior seasons. Age was the only other variable besides points that was used on a per year basis with the same nomenclature, S*n*.

Data was split into 80% training and 20% testing (holdout) sets. All model evaluation and selection was performed with 5-fold cross-validation on the training set. A single final test was done on the 20% out of sample data.

**Baseline**
- Ordinary least squares linear regression with a single feature, S1 points
- 0.642 R<sup>2</sup>
- 0.578 avg. CV R<sup>2</sup>
- 183 avg. CV RMSE

**OLS with 7 features**
- Addition of selected features above
- 0.708 R<sup>2</sup>
- 0.654 avg. CV R<sup>2</sup>
- 166 avg. CV RMSE

**OLS with 74 features**
- All non-italics features above with nationality as dummy variables
- 0.760 R<sup>2</sup>

**Lasso with scaled features**
- Regularization with optimal alpha selected 21 of 74 features
- 0.749 R<sup>2</sup>
- 0.715 avg. CV R<sup>2</sup>
- 151 avg. CV RMSE

**OLS with scaled features**
- 21 features without Lasso's coefficients
- 0.756 R<sup>2</sup>
- 0.710 avg. CV R<sup>2</sup>
- 149 avg. CV RMSE

**Out of sample OLS test**
- 21 features
- 0.687 R<sup>2</sup>
- 137 avg. CV RMSE

## Observations & Next Steps
The target distribution is highly right skewed and predictive accuracy may benefit from a log or Box-Cox transformation. The residual plots also exhibited heteroskedasticity. It is worth exploring higher degree polynomial terms to evaluate their impact on regression fit. Given the different classes of riders (domestiques, sprinters, climbers, general classification, time trial, and one-day race specialists), there is likely a classification component to points prediction. Non-linear models such as random forest and gradient boosting trees may perform better at handling these distinct groups while also avoiding issues arising from violations of OLS assumptions. Incorporating some of the unused features above and new ones such as watts/kilogram, team performance, rankings information, and data from [Dopeology](https://www.dopeology.org/) would be important to consider regardless of the algorithm chosen.
