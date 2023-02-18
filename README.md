# Zomato-Data-Analysis-and-Visualization

#import necessary packages
import pandas as pd
import matplotlib.pyplot as plt
import plotly.express as px
import seaborn as sns
import plotly.graph_objects as go

#read zomato csv file
df = pd.read_csv('zomato.csv')

df.head()

#to display the columns 
df.columns

df.info()

df.describe()

df.shape

df.isnull().sum()

## Checking if dataset contains any null

nan_values = df.isna()
nan_columns = nan_values.any()

columns_with_nan = df.columns[nan_columns].tolist()
print(columns_with_nan)

#read country code excel file
df1 = pd.read_excel('Country-Code.xlsx')
df1.head()

#merge zomato.csv file and country code excel file
df2 = pd.merge(df,df1,on='Country Code',how='left')
df2.head(2)

#to display the country currency count
df2.Country.value_counts()

"""# Add a column with rupees as the currency"""

country_currency = df2[['Country','Currency']].groupby(['Country','Currency']).size().reset_index(name='count').drop('count', axis=1, inplace=False)
country_currency.sort_values('Currency').reset_index(drop=True)

exchange_rate = 74.1

df2['Amount in Rupees'] = df2['Average Cost for two'] * exchange_rate

df2.to_csv('updated_zomato.csv', index=False)

df2

"""# plot that compares indian currency with other country’s currency"""

grouped = df2.groupby('Country Code')['Amount in Rupees'].mean().reset_index()

sns.barplot(x='Country Code', y='Amount in Rupees', data=grouped)
plt.xlabel('Country Code')
plt.ylabel('Average Cost for Two (in Rupees)')
plt.title('Comparison of Average Cost for Two Across Countries')
plt.show()

"""# Rating count in the city (based on rating test)"""

df3 = df2.groupby(['Aggregate rating','Rating color', 'Rating text']).size().reset_index().rename(columns={0:'Rating Count'})
df3
df3

"""Rating 0 — White — Not rated

Rating 1.8 to 2.4 — Red — Poor

Rating 2.5 to 3.4 — Orange — Average

Rating 3.5 to 3.9 — Yellow — Good

Rating 4.0 to 4.4 — Green — Very Good

Rating 4.5 to 4.9 — Dark Green — Excellent

Maximum restaurants seems to have gone No ratings. Let us check if these restaurants belong to some specific country.
"""

No_rating = df2[df2['Rating color']=='White'].groupby('Country').size().reset_index().rename(columns={0:'Rating Count'})
No_rating

"""India seems to have maximum unrated restaurants.

maximum business spread across restaurants in India

## Filter based on the city
"""

New_Delhi_data = df.loc[df['City'] == 'New Delhi']

# Display the filtered dataset
print(New_Delhi_data.head())

"""## Find which cuisines are costly in India"""

india_data = df2[df2['Country Code'] == 1]

costly_cuisines = india_data.groupby('Cuisines')['Amount in Rupees'].mean().sort_values(ascending=False)[:10]

costly_cuisines_chart = px.bar(x=costly_cuisines.index, y=costly_cuisines.values, title='Top 10 Costly Cuisines in India')
costly_cuisines_chart.update_layout(xaxis_title='Cuisine', yaxis_title='Average Cost for Two (Rupees)')

costly_cuisines_chart.show()

"""## Which is costlier cuisine"""

cuisine_data = df2.groupby('Cuisines')['Amount in Rupees'].mean().sort_values(ascending=False)

costlier_cuisine = cuisine_data.index[0]
print(f"The costlier cuisine is {costlier_cuisine}")

"""## Pie chart online delivery vs dine-in"""

delivery_data = df2.groupby(['Has Online delivery', 'Has Table booking']).size().reset_index(name='count')

fig = px.pie(delivery_data, values='count', names=['Online delivery - Table booking',
                                                   'Online delivery - No table booking',
                                                   'No online delivery - Table booking',
                                                   'No online delivery - No table booking'],
             title='Proportion of restaurants with online delivery and dine-in options')
fig.show()

"""## Which part of India spends more on online deliver"""

online_delivery_data = df2[df2['Has Online delivery'] == 'Yes'].groupby(['City']).mean()[['Average Cost for two']].reset_index()

top_cities = online_delivery_data.sort_values('Average Cost for two', ascending=False).head(10)

print(top_cities[['City', 'Average Cost for two']])

"""## Which part has a high living cost vs low living cost"""

avg_cost = df.groupby('Locality')['Average Cost for two'].mean()

avg_cost = avg_cost.sort_values()
avg_cost

print(avg_cost.nlargest(10))

# Print the top 10 localities with the lowest average cost
print(avg_cost.nsmallest(10))

"""## Which part of India spends more on dine-in"""

total_spent = df.groupby('City')['Average Cost for two'].sum()
print(total_spent.nlargest(10))

"""# Create a dropdown to choose the country-specific data"""

import ipywidgets as widgets
from IPython.display import display

country_dropdown = widgets.Dropdown(
    options=df2['Country'].unique(),
    value='India',
    description='Country:'
)

def update_charts(country):
  country_data = df2[df2['Country'] == country]
  cuisine_counts = country_data['Cuisines'].str.split(', ', expand=True).stack().value_counts()
  top_cuisines = cuisine_counts[:10].reset_index()
  top_cuisines.columns = ['Cuisine', 'Count']
  fig1 = px.bar(top_cuisines, x='Cuisine', y='Count', title='Top 10 Cuisines in {}'.format(country))
  fig2 = px.scatter(country_data, x='Aggregate rating', y='Average Cost for two', color='Rating color',
                      title='Average Cost for Two vs Rating in {}'.format(country))
  display(fig1)
  display(fig2)


widgets.interactive(update_charts, country=country_dropdown)
