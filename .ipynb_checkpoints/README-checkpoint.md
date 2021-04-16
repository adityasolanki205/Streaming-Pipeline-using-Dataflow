# Streaming Pipeline using DataFlow
This is one of the **Introduction to Apache Beam using Python** Repository. Here we will try to learn basics of Apache Beam to create **Streaming** pipelines. We will learn step by step how to create a streaming pipeline using [German Credit Risk](https://www.kaggle.com/uciml/german-credit). The complete process is divided into 8 parts:

1. **Generating Streaming Data**
2. **Reading Data from Pub Sub**
3. **Parsing the data**
4. **Filtering the data**
5. **Performing Type Convertion**
6. **Data wrangling**
7. **Deleting Unwanted Columns**
8. **Inserting Data in Bigquery**


## Motivation
For the last two years, I have been part of a great learning curve wherein I have upskilled myself to move into a Machine Learning and Cloud Computing. This project was practice project for all the learnings I have had. This is first of the many more to come. 
 

## Libraries/frameworks used

<b>Built with</b>
- [Apache Beam](https://beam.apache.org/documentation/programming-guide/)
- [Anaconda](https://www.anaconda.com/)
- [Python](https://www.python.org/)
- [Google DataFlow](https://cloud.google.com/dataflow)
- [Google Cloud Storage](https://cloud.google.com/storage)
- [Google Bigquery](https://cloud.google.com/bigquery)
- [Google Pub/Sub](https://cloud.google.com/pubsub)

## Cloning Repository

```bash
    # clone this repo:
    git clone https://github.com/adityasolanki205/Streaming-Pipeline-using-Dataflow.git
```

## Pipeline Construction

Below are the steps to setup the enviroment and run the codes:

1. **Setup**: First we will have to setup free google cloud account which can be done [here](https://cloud.google.com/free). Then we need to Download the data from [German Credit Risk](https://www.kaggle.com/uciml/german-credit).

2. **Cloning the Repository to Cloud SDK**: We will have to copy the repository on Cloud SDK using below command:

```bash
    # clone this repo:
    git clone https://github.com/adityasolanki205/Batch-Processing-Pipeline-using-DataFlow.git
```

3. **Generating Streaming Data**: We need to generate streaming data that can be published to Pub Sub. Then those messages will be picked to be processed by the pipeline. To generate data we will use **random()** library to create input messages. Using the generating_data.py we will be able to generate random data in the required format. This generated data will be published to Pub/Sub using publish_to_pubsub.py. Here we will use PublisherClient object, add the path to the topic using the topic_path method and call the publish_to_pubsub() function while passing the topic_path and data.

```python
    import random

    LINE ="""   {Existing_account} 
                {Duration_month} 
                {Credit_history} 
                {Purpose} 
                {Credit_amount} 
                .....
                {Classification}"""

    def generate_log():
        existing_account = ['B11','A12','C14',
                            'D11','E11','A14',
                            'G12','F12','A11',
                            'NULL','H11','I11',
                            'J14','K14','L11',
                            'A13'
                           ]
        Existing_account = random.choice(existing_account)
    
        duration_month = []
        for i  in range(6, 90 , 3):
            duration_month.append(i)
        Duration_month = random.choice(duration_month)
        ....
        
        classification = ['NULL',
        '1',
        '2']
        Classification = random.choice(classification)
        log_line = LINE.format(
            Existing_account=Existing_account,
            Duration_month=Duration_month,
            Credit_history=Credit_history,
            Purpose=Purpose,
            ...
            Classification=Classification
        )

        return log_line

```

3. **Reading Data from Pub Sub**: Now we will start reading data from Pub sub to start the pipeline . The data is read using **beam.io.ReadFromPubSub()**. Here we will just read the input message by providing the TOPIC and the output is decoded which was encoded while generating the data. 

```python
    def run(argv=None, save_main_session=True):
        parser = argparse.ArgumentParser()
        parser.add_argument(
          '--input',
          dest='input',
          help='Input file to process')
        parser.add_argument(
          '--output',
          dest='output',
          default='../output/result.txt',
          help='Output file to write results to.')
        known_args, pipeline_args = parser.parse_known_args(argv)
        options = PipelineOptions(pipeline_args)
        TOPIC ="projects/trusty-field-283517/topics/german_credit_data"
        with beam.Pipeline(options=PipelineOptions()) as p:
            encoded_data = ( p 
                             | 'Read data' >> beam.io.ReadFromPubSub(topic=TOPIC).with_output_types(bytes) 
                           )
                    data = ( encoded_data
                             | 'Decode' >> beam.Map(lambda x: x.decode('utf-8') ) 
                           ) 
    if __name__ == '__main__':
        run()
``` 

4. **Parsing the data**: After reading the input from Pub-Sub we will split the data using split(). Data is segregated into different columns to be used in further steps. We will **ParDo()** to create a split function. The output of this step is present in SplitPardo text file.

```python
    class Split(beam.DoFn):
        #This Function Splits the Dataset into a dictionary
        def process(self, element): 
            Existing_account,
            Duration_month,
            Credit_history,
            Purpose,
            Credit_amount,
            Saving,
            Employment_duration,
            Installment_rate,
            Personal_status,
            Debtors,
            Residential_Duration,
            Property,
            Age,
            Installment_plans,
            Housing,
            Number_of_credits
            Job,
            Liable_People,
            Telephone,
            Foreign_worker,
            Classification = element.split(' ')
        return [{
            'Existing_account': str(Existing_account),
            'Duration_month': str(Duration_month),
            'Credit_history': str(Credit_history),
            'Purpose': str(Purpose),
            'Credit_amount': str(Credit_amount),
            'Saving': str(Saving),
            'Employment_duration':str(Employment_duration),
            'Installment_rate': str(Installment_rate),
            'Personal_status': str(Personal_status),
            'Debtors': str(Debtors),
            'Residential_Duration': str(Residential_Duration),
            'Property': str(Property),
            'Age': str(Age),
            'Installment_plans':str(Installment_plans),
            'Housing': str(Housing),
            'Number_of_credits': str(Number_of_credits),
            'Job': str(Job),
            'Liable_People': str(Liable_People),
            'Telephone': str(Telephone),
            'Foreign_worker': str(Foreign_worker),
            'Classification': str(Classification)
        }]
    def run(argv=None, save_main_session=True):
        ...
        with beam.Pipeline(options=PipelineOptions()) as p:
            encoded_data = ( p 
                     | 'Read data' >> beam.io.ReadFromPubSub(topic=TOPIC).with_output_types(bytes))
            data   = ( encoded_data
                     | 'Decode' >> beam.Map(lambda x: x.decode('utf-8'))) 
            parsed_data = (data 
                     | 'Parsing Data' >> beam.ParDo(Split())
                     | 'Writing output' >> beam.io.WriteToText(known_args.output))
            
    if __name__ == '__main__':
        run()
``` 

5. **Filtering the data**: Now we will clean the data by removing all the rows having Null values from the dataset. We will use **Filter()** to return only valid rows with no Null values. Output of this step is saved in the file named Filtered_data.

```python
    ...
    def Filter_Data(data):
    #This will remove rows the with Null values in any one of the columns
        return data['Purpose'] !=  'NULL' 
        and len(data['Purpose']) <= 3  
        and data['Classification'] !=  'NULL' 
        and data['Property'] !=  'NULL' 
        and data['Personal_status'] != 'NULL' 
        and data['Existing_account'] != 'NULL' 
        and data['Credit_amount'] != 'NULL' 
        and data['Installment_plans'] != 'NULL'
    ...
    def run(argv=None, save_main_session=True):
        ...
        with beam.Pipeline(options=PipelineOptions()) as p:
            encoded_data = ( p 
                     | 'Read data' >> beam.io.ReadFromPubSub(topic=TOPIC).with_output_types(bytes))
            data   = ( encoded_data
                     | 'Decode' >> beam.Map(lambda x: x.decode('utf-8'))) 
            parsed_data = (data 
                     | 'Parsing Data' >> beam.ParDo(Split()))
            filtered_data = (parsed_data
                     | 'Filtering Data' >> beam.Filter(Filter_Data)          
                     | 'Writing output' >> beam.io.WriteToText(known_args.output))

    if __name__ == '__main__':
        run()
```

6. **Performing Type Convertion**: After Filtering we will convert the datatype of numeric columns from String to Int or Float datatype. Here we will use **Map()** to apply the Convert_Datatype(). The output of this step is saved in Convert_datatype text file.

```python
    ... 
    def Convert_Datatype(data):
        #This will convert the datatype of columns from String to integers or Float values
        data['Duration_month'] = int(data['Duration_month']) if 'Duration_month' in data else None
        data['Credit_amount'] = float(data['Credit_amount']) if 'Credit_amount' in data else None
        data['Installment_rate'] = int(data['Installment_rate']) if 'Installment_rate' in data else None
        data['Residential_Duration'] = int(data['Residential_Duration']) if 'Residential_Duration' in data else None
        data['Age'] = int(data['Age']) if 'Age' in data else None
        data['Number_of_credits'] = int(data['Number_of_credits']) if 'Number_of_credits' in data else None
        data['Liable_People'] = int(data['Liable_People']) if 'Liable_People' in data else None
        data['Classification'] =  int(data['Classification']) if 'Classification' in data else None
    ...
    def run(argv=None, save_main_session=True):
        ...
        with beam.Pipeline(options=PipelineOptions()) as p:
            encoded_data = ( p 
                     | 'Read data' >> beam.io.ReadFromPubSub(topic=TOPIC).with_output_types(bytes))
            data   = ( encoded_data
                     | 'Decode' >> beam.Map(lambda x: x.decode('utf-8'))) 
            parsed_data = (data 
                     | 'Parsing Data' >> beam.ParDo(Split()))
            filtered_data = (parsed_data
                     | 'Filtering Data' >> beam.Filter(Filter_Data))
            Converted_data = (filtered_data
                     | 'Convert Datatypes' >> beam.Map(Convert_Datatype)
                     | 'Writing output' >> beam.io.WriteToText(known_args.output))

    if __name__ == '__main__':
        run()
```

7. **Data wrangling**: Now we will do some data wrangling to make some more sense of the data in some columns. For Existing_account contain 3 characters. First character is an Aplhabet which signifies Month of the year and next 2 characters are numeric which signify days. So here as well we will use Map() to wrangle data. The output of this dataset is present by the name DataWrangle.

```python
    ... 
    def Data_Wrangle(data):
    #Here we perform data wrangling where Values in columns are converted to make more sense
        Month_Dict = {
        'A':'January',
        'B':'February',
        'C':'March',
        'D':'April',
        'E':'May',
        'F':'June',
        'G':'July',
        'H':'August',
        'I':'September',
        'J':'October',
        'K':'November',
        'L':'December'
        }
        existing_account = list(data['Existing_account'])
        for i in range(len(existing_account)):
            month = Month_Dict[existing_account[0]]
            days = int(''.join(existing_account[1:]))
            data['Month'] = month
            data['days'] = days
        purpose = list(data['Purpose'])
        for i in range(len(purpose)):
            file_month = Month_Dict[purpose[0]]
            version = int(''.join(purpose[1:]))
            data['File_Month'] = file_month
            data['Version'] = version
        return data
    ...
    def run(argv=None, save_main_session=True):
        ...
        with beam.Pipeline(options=PipelineOptions()) as p:
            encoded_data = ( p 
                     | 'Read data' >> beam.io.ReadFromPubSub(topic=TOPIC).with_output_types(bytes))
            data   = ( encoded_data
                     | 'Decode' >> beam.Map(lambda x: x.decode('utf-8'))) 
            parsed_data = (data 
                     | 'Parsing Data' >> beam.ParDo(Split()))
            filtered_data = (parsed_data
                     | 'Filtering Data' >> beam.Filter(Filter_Data))
            Converted_data = (filtered_data
                     | 'Convert Datatypes' >> beam.Map(Convert_Datatype))
            Wrangled_data = (Converted_data
                     | 'Wrangling Data' >> beam.Map(Data_Wrangle)                  
                     | 'Writing output' >> beam.io.WriteToText(known_args.output))

    if __name__ == '__main__':
        run()
```

8. **Delete Unwanted Columns**: After converting certain columns to sensable data we will remove redundant columns from the dataset. Output of this is present with the file name Delete_Unwanted_Columns text file.

```python
    ...
    def Del_Unwanted(data):
        #Here we delete redundant columns
        del data['Purpose']
        del data['Existing_account']
        return data
    ...
    def run(argv=None, save_main_session=True):
        ...
        with beam.Pipeline(options=PipelineOptions()) as p:
            encoded_data = ( p 
                     | 'Read data' >> beam.io.ReadFromPubSub(topic=TOPIC).with_output_types(bytes))
            data   = ( encoded_data
                     | 'Decode' >> beam.Map(lambda x: x.decode('utf-8'))) 
            parsed_data = (data 
                     | 'Parsing Data' >> beam.ParDo(Split()))
            filtered_data = (parsed_data
                     | 'Filtering Data' >> beam.Filter(Filter_Data))
            Converted_data = (filtered_data
                     | 'Convert Datatypes' >> beam.Map(Convert_Datatype))
            Wrangled_data = (Converted_data
                     | 'Wrangling Data' >> beam.Map(Data_Wrangle))    
            Cleaned_data = (Wrangled_data
                     | 'Delete Unwanted Columns' >> beam.Map(Del_Unwanted)                 
                     | 'Writing output' >> beam.io.WriteToText(known_args.output))

    if __name__ == '__main__':
        run()    
```

9. **Inserting Data in Bigquery**: Final step in the Pipeline it to insert the data in Bigquery. To do this we will use **beam.io.WriteToBigQuery()** which requires Project id and a Schema of the target table to save the data. 

```python
    import apache_beam as beam
    from apache_beam.options.pipeline_options import PipelineOptions
    import argparse
    
    SCHEMA = 
    '
    Duration_month:INTEGER,
    Credit_history:STRING,
    Credit_amount:FLOAT,
    Saving:STRING,
    Employment_duration:STRING,
    Installment_rate:INTEGER,
    Personal_status:STRING,
    Debtors:STRING,
    Residential_Duration:INTEGER,
    Property:STRING,
    Age:INTEGER,
    Installment_plans:STRING,
    Housing:STRING,
    Number_of_credits:INTEGER,
    Job:STRING,
    Liable_People:INTEGER,
    Telephone:STRING,
    Foreign_worker:STRING,
    Classification:INTEGER,
    Month:STRING,
    days:INTEGER,
    File_Month:STRING,
    Version:INTEGER
    '
    ...
    def run(argv=None, save_main_session=True):
        ...
        parser.add_argument(
          '--project',
          dest='project',
          help='Project used for this Pipeline')
        ...
        PROJECT_ID = known_args.project
        with beam.Pipeline(options=PipelineOptions()) as p:
            encoded_data = ( p 
                     | 'Read data' >> beam.io.ReadFromPubSub(topic=TOPIC).with_output_types(bytes))
            data   = ( encoded_data
                     | 'Decode' >> beam.Map(lambda x: x.decode('utf-8'))) 
            parsed_data = (data 
                     | 'Parsing Data' >> beam.ParDo(Split()))
            filtered_data = (parsed_data
                     | 'Filtering Data' >> beam.Filter(Filter_Data))
            Converted_data = (filtered_data
                     | 'Convert Datatypes' >> beam.Map(Convert_Datatype))
            Wrangled_data = (Converted_data
                     | 'Wrangling Data' >> beam.Map(Data_Wrangle))    
            Cleaned_data = (Wrangled_data
                     | 'Delete Unwanted Columns' >> beam.Map(Del_Unwanted)  
            output =( Cleaned_data      
                     | 'Writing to bigquery' >> beam.io.WriteToBigQuery(
                       '{0}:GermanCredit.GermanCreditTable'.format(PROJECT_ID),
                       schema=SCHEMA,
                       write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND)
                    )

    if __name__ == '__main__':
        run()        
```

## Tests
To test the code we need to do the following:

    1. Copy the repository in Cloud SDK using below command:
    git clone https://github.com/adityasolanki205/Streaming-Pipeline-using-DataFlow.git
    
    2. Create a Storage Bucket in us-east1 with 2 separate folders temp and stream
    
    3. Create a Dataset in us-east1 by the name GermanCredit
    
    4. Create a table in GermanCredit dataset by the name GermanCreditTable
    
    5. Create Pub Sub Topic by the name german_credit_data
    
    6. Install Apache Beam on the SDK using below command
    sudo pip3 install apache_beam[gcp]
    
    7. Run the command and see the magic happen:
     python3 streaming-pipeline.py \
     --runner DataFlowRunner \
     --project trusty-field-283517 \
     --temp_location gs://streaming_pipeline_testing/temp \
     --staging_location gs://streaming_pipeline_testing/stream \
     --region us-east1 \
     --job_name germanstreaminganalysis \
     --streaming 
     
    8. Open one more tab in cloud SDK and run below command 
    python3 publish_to_pubsub.py

## Credits
1. Akash Nimare's [README.md](https://gist.github.com/akashnimare/7b065c12d9750578de8e705fb4771d2f#file-readme-md)
2. [Apache Beam](https://beam.apache.org/documentation/programming-guide/#triggers)
3. [Building Data Processing Pipeline With Apache Beam, Dataflow & BigQuery](https://towardsdatascience.com/apache-beam-pipeline-for-cleaning-batch-data-using-cloud-dataflow-and-bigquery-f9272cd89eba)
4. [Let’s Build a Streaming Data Pipeline](https://towardsdatascience.com/lets-build-a-streaming-data-pipeline-e873d671fc57)
