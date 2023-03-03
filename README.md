# -Base_File-Logistics-

1. Remove Cities which are classified as Other or have no values from the base file
import pandas as pd
df = pd.read_excel(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\base_file.xlsx')
df = df[(df['city'] != 'Other')]
df.to_excel(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\base_file.xlsx', index=False)

2.With the help of the Logistics table, bring up the logistics cost for a logistic provider in the base file wherever possible
import pandas as pd
base_file = pd.read_csv(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\base_file.csv')
logistics_file = pd.read_csv(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\Logistics file.csv')
merged_file = pd.merge(base_file, logistics_file[['Logistics provider', 'Logistic cost']],
                       left_on='logistic_provider_name', right_on='Logistics provider', how='left')
merged_file.drop('Logistics provider', axis=1, inplace=True)
merged_file.to_csv(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\merged_file.csv', index=False)
print('merged_file')

3. Create a column to classify all orders in base file based on the following criteria
      i) Package type is HA and Sample type is other (To be classified as HA)
      ii) Package type is either Lab-Test/RTPCR and sample type is other (To be classified as Labs)
      iii) Package type is either Lab-Test/RTPCR and sample type is imaging (To be classified as Radiography)
      
      import pandas as pd
base_file = pd.read_excel(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\base_file.xlsx')
def classify_order(row):
    if row['package_type'] == 'HA' and row['sample_type'] != 'other':
        return 'Unknown'
    elif row['package_type'] in ['Lab-Test', 'RTPCR'] and row['sample_type'] != 'other':
        return 'Labs'
    elif row['package_type'] in ['Lab-Test', 'RTPCR'] and row['sample_type'] == 'imaging':
        return 'Radiography'
    else:
        return 'Unknown'
base_file['Order_Type'] = base_file.apply(classify_order, axis=1)
base_file.to_excel(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\base_file_updated.xlsx', index=False)

4. Create and export a new table showing the total for booking amounts and/or logistic costs and/or count of orders where the base data has been grouped based on
      i)the classification created in the above step
      ii)payment mode
      
  import pandas as pd
merged_file = pd.read_csv(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\merged_file.csv')
merged_file['classification'] = merged_file['actual_city_name'].apply(lambda x: 'Metro' if x in ['Delhi', 'Mumbai', 'Kolkata', 'Chennai'] else 'Non-Metro')
pivot_table = merged_file.pivot_table(index=['classification', 'payment_mode'], values=['booking_amount', 'Logistic cost', 'order_id'], aggfunc={'booking_amount': 'sum', 'Logistic cost': 'sum', 'order_id': 'count'})
pivot_table.to_csv(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\pivot_table.csv')
   
5. If possible, provide any graphical insights (Bar Charts, Pie Charts, Histograms and so on)from the base data or the created table in the above step

import pandas as pd
import matplotlib.pyplot as plt
pivot_table = pd.read_csv('pivot_table.csv')
pivot_table.plot(kind='bar', x=['payment_mode', 'classification'], y='booking_amount', figsize=(10, 6))
plt.title('Total Booking Amount by Payment Mode and Classification')
plt.xlabel('Payment Mode and Classification')
plt.ylabel('Total Booking Amount')
plt.show()
pivot_table.groupby('payment_mode')['order_id'].sum().plot(kind='pie', figsize=(8, 8), autopct='%1.1f%%')
plt.title('Count of Orders by Payment Mode')
plt.ylabel('')
plt.show()

6. Try to split the base data into as many months available under order date and export it all to a single file where the sheet is labelled with the month name and the data only contains relevant info for that particular month.
import pandas as pd

base_file = pd.read_excel(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\base_file.xlsx')
logistics_file = pd.read_excel(r'C:\Users\ADMIN\OneDrive\Desktop\Logistics & Base file\Logistics file.xlsx')
base_file = base_file[base_file['city'].isin(['Other', None])]
base_file = base_file.merge(logistics_file['logistic_provider', 'Logistic_cost'], on='logistic_provider_name', how='left')
base_file['order_class'] = None
base_file.loc[(base_file['package_type'] == 'HA') & (base_file['sample_type'] == 'other'), 'order_class'] = 'HA'
base_file.loc[(base_file['package_type'].isin(['Lab-Test', 'RTPCR'])) & (base_file['sample_type'] == 'other'), 'order_class'] = 'Labs'
base_file.loc[(base_file['package_type'].isin(['Lab-Test', 'RTPCR'])) & (base_file['sample_type'] == 'imaging'), 'order_class'] = 'Radiography'
grouped_table = base_file.groupby(['order_class', 'payment_mode']).agg({'booking_amount': 'sum', 'Logistic_cost': 'sum', 'order_id': 'count'}).reset_index()
grouped_table.to_excel('grouped_table.xlsx', index=False)
for month in base_file['order_date'].dt.month.unique():
    month_file = base_file[base_file['order_date'].dt.month == month]
    month_file.to_excel(f'{month}_file.xlsx', index=False)














