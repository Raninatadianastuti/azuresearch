# azuresearch

This demo shows how to create an Azure Search Instance and index JSON files stored in an associated Blob Storage account.

1. The ARM template will deploy both Search and Blob Storage using the button below.  Note - keep the search and blob names short as there is a maximum length of 24 characters and the ARM template will add a guid to the end of the value you provide. [![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://azuredeploy.net/)
2. Create a container in the blob storage account called "allblobs" (you can call it something different, but you'll have to change the REST calls to reflect it), and upload a sample file to it. I used a publicly available [food hygeine dataset](https://data.gov.uk/dataset/uk-food-hygiene-rating-data-yorkshire-and-humberside-food-standards-agency/resource/b290ee03-1405-4b90-ae63-2ae09d8c7791) from DATA.GOV.UK.  The data is available in XML format so I used a simple XML to JSON transformation.  I recommend using [Azure Storage Explorer](http://storageexplorer.com/) for uploading the file.
2. Next, you need to create a search index.

```
POST https://<mysearch>.search.windows.net/indexes?api-version=2015-02-28-Preview 
Content-Type: application/json 
api-key: <apikey>

{
 "name": "default",  
 "fields": [
  {"name": "id", "type": "Edm.String", "key": true, "searchable": false},
  {"name": "businessname", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true},
 ]
}

```

3. Then, add a data source to point to your blob storage account.

```
POST https://<mysearch>.search.windows.net/datasources?api-version=2015-02-28-Preview 
Content-Type: application/json 
api-key: <apikey>

{
    "name" : "blob-datasource",
    "type" : "azureblob",
    "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<blobname>;AccountKey=<blobaccountkey>" },
    "container" : { "name" : "allblobs" }
}   

```

4. Once the data source has been added, then you need to add an indexer, which maps the data source to the index.

```
{
  "name" : "bob-json-indexer",
  "dataSourceName" : "blob-datasource",
  "targetIndexName" : "default",
  "schedule" : { "interval" : "PT2H" },
  "parameters" : { "configuration" : { "parsingMode" : "jsonArray", "documentRoot" : "/FHRSEstablishment/EstablishmentCollection/EstablishmentDetail" } },
  "fieldMappings" : [
    { "sourceFieldName" : "/FHRSID", "targetFieldName" : "id" },
    { "sourceFieldName" : "/BusinessName", "targetFieldName" : "businessname" }
    ]
}

```

5. Test!


