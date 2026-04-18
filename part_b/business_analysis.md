Part B: Business Case Analysis
B1. Problem Formulation
(a) Machine Learning Formulation

Target Variable: items_sold (The total number of items sold per store per month).

Candidate Input Features: * promotion_type (Flat Discount, BOGO, etc.)

store_size (Small, Medium, Large)

location_type (Urban, Semi-urban, Rural)

is_weekend / is_festival (Temporal flags)

competition_density (Local market saturation)

month / season (Extracted from date)

ML Problem Type: This is a Regression problem because the target variable (items_sold) is a continuous numerical value. We are predicting a quantity rather than a category.

(b) Target Variable Selection
Using items sold (sales volume) is more reliable than sales revenue because revenue is directly influenced by the discounts themselves. For example, a "Flat 50% Discount" might increase the number of items sold significantly but could show lower revenue compared to a "Loyalty Points" promotion. Sales volume reflects the true consumer demand and promotion effectiveness without the noise of price fluctuations.

Broader Principle: This illustrates the importance of selecting a target variable that is a direct proxy for the business objective (moving inventory) rather than one influenced by external confounding factors like price elasticity or margin changes.

(c) Alternative Modelling Strategy
Instead of one global model, I propose a Clustered or Segmented Modelling Strategy. We can build separate models for different location_type segments (e.g., one for Urban stores, one for Rural).

Justification: Customer behavior in urban areas (convenience-driven) differs vastly from rural areas (price-sensitive). A single model might "average out" these nuances, whereas segmented models can learn specific patterns for each environment, leading to more accurate local recommendations.

B2. Data and EDA Strategy
(a) Data Integration and Grain

Joining Process: I would perform a Left Join starting with the calendar table, joining transactions on the date, then joining store attributes on store_id, and finally promotion details on the promotion ID/Type.

Grain of Data: The final grain will be One row per Store per Month.

Aggregations: Since raw transactions are at the item level, I would sum the items_sold and revenue per store per month. I would also calculate the average footfall and count the number of promotion days in that month.

(b) EDA Plan

Promotion Type vs. Items Sold (Boxplot): To identify which promotion generally has the highest median sales and see the variance.

Time-Series Line Plot: To check for seasonality (e.g., do sales spike every December regardless of promotion?).

Location Type vs. Promotion Response (Grouped Bar Chart): To see if BOGO works better in Urban stores while Flat Discounts work better in Rural ones.

Correlation Heatmap: To identify if competition_density or store_size has a strong linear relationship with sales.

(c) Handling Promotion Imbalance
When 80% of data has "No Promotion," the model may become biased toward predicting "average" sales and fail to capture the "lift" caused by a promotion.

Strategy: I would use Under-sampling of the "No Promotion" days to balance the dataset or create a Relative Lift target variable (Items sold with promo / Average items sold without promo) to focus specifically on the impact of the marketing intervention.

B3. Model Evaluation and Deployment
(a) Train-Test Split and Metrics

Split Strategy: A Time-based (Temporal) Split is required. For example, use the first 30 months for training and the last 6 months for testing.

Why not Random? Random splits ignore the chronological order, allowing the model to "leak" future information (like holiday trends) into the past, resulting in over-optimistic performance.

Metrics:

MAE (Mean Absolute Error): Easy to explain to business (e.g., "The model is off by 20 items on average").

RMSE (Root Mean Squared Error): Useful if large errors are particularly costly for inventory management.

(b) Communicating Recommendations
Using Feature Importance (like SHAP values), I would show the marketing team that in December, the is_festival feature is the dominant driver, making "Loyalty Points" more effective as people are already shopping. In March, a "Flat Discount" might be recommended because the is_festival value is 0, and the model recognizes that a price drop is necessary to trigger purchases during a "slow" month.

(c) Deployment and Monitoring

Deployment: Save the trained pipeline as a .pkl or .joblib file. Use a cloud function (like AWS Lambda or a local Cron job) to trigger the model on the 1st of every month.

Preparation: New monthly data (Store info + upcoming Calendar events) will be fed into the pipeline.

Monitoring: I would monitor Concept Drift. If the distribution of items_sold changes significantly (e.g., due to an economic recession), the model's error (MAE) will increase. Once the error crosses a predefined threshold, a trigger will alert the team to retrain the model with the most recent data.