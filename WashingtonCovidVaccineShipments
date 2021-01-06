import pandas as pd
import io
import requests
import matplotlib.pyplot as plt

#CONNECT TO CDC API AND IMPORT CONTENT TO A DATAFRAME

ModernaURL="https://data.cdc.gov/resource/b7pe-5nws.csv"
ModernaContent = requests.get(ModernaURL).content
ModernaDF = pd.read_csv(io.StringIO(ModernaContent.decode('utf-8')))

PfizerURL="https://data.cdc.gov/resource/saz5-9hgg.csv"
PfizeraContent = requests.get(PfizerURL).content
PfizerDF = pd.read_csv(io.StringIO(PfizeraContent.decode('utf-8')))

#FILTER DATAFRAME FOR ONLY WASHINGTON STATE
ModernaDF = ModernaDF[ModernaDF['jurisdiction'].str.contains("Washington")]
ModernaDF = ModernaDF.replace(',','', regex=True)

PfizerDF = PfizerDF[PfizerDF['jurisdiction'].str.contains("Washington")]
PfizerDF = PfizerDF.replace(',','', regex=True)

#RESET INDEX SO THAT THE FIRST ROW NUMBER IS 0
ModernaDF = ModernaDF.reset_index()
PfizerDF = PfizerDF.reset_index()
#CREATE DYNAMIC WEEK NUMBER COLUMN LIST (THE DATA IS SUCH THAT FOR EACH NEW WEEK, THE CDC ADDS A NEW COLUMN)
column_index_Moderna = ModernaDF.columns
column_index_Moderna = column_index_Moderna.drop(['index','jurisdiction', 'hhs_region','total_moderna_allocation_first_dose_shipments','total_allocation_moderna_second_dose_shipments']) 

column_index_Pfizer = PfizerDF.columns
column_index_Pfizer = column_index_Pfizer.drop(['index','jurisdiction', 'hhs_region','total_pfizer_allocation_first_dose_shipments','total_allocation_pfizer_second_dose_shipments']) 


#CONVERT WEEK NUMBER COLUMNS FROM STRINGS TO NUMERIC DATA TYPES
ModernaDF[column_index_Moderna] = ModernaDF[column_index_Moderna].apply(pd.to_numeric, errors='coerce')
PfizerDF[column_index_Pfizer] = PfizerDF[column_index_Pfizer].apply(pd.to_numeric, errors='coerce')

#CREATE SUBSET OF DATAFRAME FOR CHART
ModernaShipments = ModernaDF.drop(columns = ['index','jurisdiction','hhs_region','total_moderna_allocation_first_dose_shipments','total_allocation_moderna_second_dose_shipments'])
PfizerShipments = PfizerDF.drop(columns = ['index','jurisdiction','hhs_region','total_pfizer_allocation_first_dose_shipments','total_allocation_pfizer_second_dose_shipments']) 


#PREPROCESS COLUMN HEADER TEXT FOR CLARITY
ModernaShipments.columns = ModernaShipments.columns.str.lstrip('first_doses_allocated_second_dose_shipment_week_of_')
ModernaShipments = ModernaShipments.groupby(ModernaShipments.columns, axis=1, sort = False).sum()
ModernaShipments.columns = ModernaShipments.columns.str.replace("_", "-")
    #add 0 values for the two weeks that Pfizer was approved but Moderna was not
ModernaShipments.insert(loc=0, column='12-17', value=0)
ModernaShipments.insert(loc=0, column='12-14', value=0)

PfizerShipments.columns = PfizerShipments.columns.str.lstrip('first_doses_allocated_second_dose_shipment_week_of_')
PfizerShipments = PfizerShipments.groupby(PfizerShipments.columns, axis=1, sort = False).sum()
PfizerShipments.columns = PfizerShipments.columns.str.replace("_", "-")
headers = list(PfizerShipments.columns)
    #change position of 12-17 and 12-21. These two columns are stored in reverse order in the CDC database
headers[1],headers[2] = headers[2],headers[1]
PfizerShipments = PfizerShipments[headers]

#SELECT FIRST ROW OF VALUES (WHICH IS THE ONLY ROW OF VALUES) FROM EACH DATAFRAME FOR THE CHART
ModernaSeries = ModernaShipments.iloc[0]
PfizerSeries = PfizerShipments.iloc[0]

ModernaSeries = ModernaSeries / 1000
PfizerSeries = PfizerSeries / 1000

#CREATE VALUES FOR MATPLOTLIB CHART
    #total vaccines shipped
TotalModerna = int(ModernaDF['total_moderna_allocation_first_dose_shipments'][0]) + int(ModernaDF['total_allocation_moderna_second_dose_shipments'][0])
TotalPfizer =  int(PfizerDF['total_pfizer_allocation_first_dose_shipments'][0]) + int(PfizerDF['total_allocation_pfizer_second_dose_shipments'][0])
TotalDosesSent = str((TotalModerna + TotalPfizer)/1000)

plt.plot(ModernaSeries, label = "Moderna")
plt.plot(PfizerSeries, label = "Pfizer")
plt.xticks(range(0,len(ModernaShipments.columns)),ModernaShipments.columns)
plt.xticks(range(0,len(PfizerShipments.columns)),PfizerShipments.columns)
plt.title('Washington State Covid-19 Vaccine Doses Shipped (thousands)')
plt.ylabel("Doses Sent (thousands)")
plt.xlabel("Week")
plt.legend()
plt.text(1.5, 4, 'Total Doses Sent to WA (thousands): ' + TotalDosesSent)

for i in range(len(ModernaSeries)):
    pos = ModernaSeries[i]
    string = '{:}'.format(pos)
    plt.text(i,pos,string,ha='center',color='black')

for i in range(len(PfizerSeries)):
    pos = PfizerSeries[i]
    string = '{:}'.format(pos)
    plt.text(i,pos,string,ha='center',color='black')

#SHOW THE PLOT
plt.show()

#SAVE TO PNG
