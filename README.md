# Pyspark-Diabetes-Dataset

Using the Diabetes dataset from Kaggle, I delved into data analysis, preprocessing, and machine learning. Key steps included reading and exploring the dataset, renaming columns for clarity, adding a diabetes indicator column, filtering data for insights, and using Vector Assembler for ML preparation. I trained a linear regression model to predict diabetes outcomes, revealing valuable insights. Next, I plan to visualize results and explore more ML algorithms.

!pip install pyspark
import pyspark
from pyspark.sql import SparkSession
spark= SparkSession.builder.appName('PySpark').getOrCreate()
df_pyspark=spark.read.csv('/content/diabetes.csv')
df_pyspark.columns
df_pyspark=spark.read.csv('/content/diabetes.csv',header=True)
df_pyspark
df_pyspark=spark.read.csv('/content/diabetes.csv',header=True,inferSchema=True)
df_pyspark
df_pyspark.printSchema()
type(df_pyspark)
df_pyspark.head(3)
df_pyspark.select(['DiabetesPedigreeFunction','Outcome']).show()
df_pyspark.drop('Pregnancies').show()
df_pyspark.withColumnsRenamed({'DiabetesPedigreeFunction':'Diabetes Percentage','Pregnancies':'No.of Pregnancies'}).show()

from pyspark.sql.functions import when
df_pyspark.withColumn('Results',when(df_pyspark['Outcome'] == 1, "Yes").otherwise("No")).show()
df_pyspark.filter((df_pyspark['Outcome']==1)).select(['Age',"BloodPressure","Outcome"]).show()    ##This implies that age 25 and above have the problem of diabetes.

## **PySPARK GroupBy and Aggregate Functions**
df_pyspark.groupBy('Outcome').count().show()
df_pyspark.groupBy('BloodPressure').count().show()
df_pyspark.groupBy().min('DiabetesPedigreeFunction').show()
df_pyspark.groupBy().max('DiabetesPedigreeFunction').show()
df_pyspark.groupBy().sum('Pregnancies').show()

**Implementing Vector Assembler on the Diabetes Dataset**
from pyspark.ml.feature import VectorAssembler
train= SparkSession.builder.appName('Vectorassembler').getOrCreate()
training_dt=train.read.csv('/content/diabetes.csv',header=True,inferSchema=True)
training_dt.show()
training_dt.columns
featureassembler=VectorAssembler(inputCols=['Insulin','Age'],outputCol='Independent Features')
output=featureassembler.transform(training_dt)
output.show()
final_data=output.select("Independent Features","Outcome")
final_data.show()
from pyspark.ml.regression import LinearRegression
train_data,test_data=final_data.randomSplit([0.45,0.55])                                   ##train_test split
regressor=LinearRegression(featuresCol='Independent Features',labelCol='Outcome')
regressor=regressor.fit(train_data)
regressor.coefficients
regressor.intercept
pred_results=regressor.evaluate(test_data)
pred_results.predictions.show()

**Plotting Graph in PySPARK**
x = pred_results.predictions.select("Independent Features").rdd.flatMap(lambda x: x).collect()
y = pred_results.predictions.select("prediction").rdd.flatMap(lambda x: x).collect()
x_values = [vector[0] for vector in x]
import matplotlib.pyplot as plt
plt.scatter(x_values, y)
plt.xlabel("Independent Features")
plt.ylabel("Predictions")
plt.show()







