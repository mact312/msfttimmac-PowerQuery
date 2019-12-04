//////////////////////////////////////////////////////////////////////////////////////////////
// M QUERY: Connect to Azure Log Analytics using OAUTH2 and a Client Secret
// ---------------------------------------------------------------------------------------------------------------
// Data Source: Set to Anonymous
// Change the Unique IDs
// 
// - %TENANT_ID% = Tenant_ID
// - %Application_Client_ID% = Application Client ID
// - %CLIENT_SECRET% = client secret from application
// - %WORKSPACE_ID% Workspace ID
 
//Get the OAUTH2 Token
let 
 GetJson = Web.Contents("https://login.microsoftonline.com/%TENANT_ID%/oauth2/token",
     [
         Content = Text.ToBinary("grant_type=client_credentials&scope=https://api.loganalytics.io&resource=https://api.loganalytics.io&client_id=%APPLICATION_CLIENT_ID%&client_secret=%CLIENT_SECRET%") 
     ]
),
 
// Once we have the token, we then take the variable and push it into a JSON Document
// The JSON Response will have information and a piece of that will be the Access_Token
// From within the JSON Document, we grab the Access_Token item
FormatAsJson = Json.Document(GetJson),
AccessToken = FormatAsJson[access_token],

// Now that we have the Access_Token, let's create the Token in the correct format
AccessTokenHeader = "Bearer " & AccessToken,

 // Now that we have the Access_Token, let's pass it to authenticate to get query
  Source = Json.Document(Web.Contents("https://api.loganalytics.io/v1/workspaces/%WORKSPACE_ID%/query",
     [
     Headers = [#"Authorization"=AccessTokenHeader],
     Query=[#"query"="DnsEvents",#"x-ms-app"="OmsAnalyticsPBI",#"timespan"="P1D",#"prefer"="ai.response-thinning=true"],
     Timeout=#duration(0,0,4,0)    
      ]
)),
TypeMap = #table(
{ "AnalyticsTypes", "Type" }, 
{ 
{ "string",   Text.Type },
{ "int",      Int32.Type },
{ "long",     Int64.Type },
{ "real",     Double.Type },
{ "timespan", Duration.Type },
{ "datetime", DateTimeZone.Type },
{ "bool",     Logical.Type },
{ "guid",     Text.Type },
{ "dynamic",  Text.Type }
}),
DataTable = Source[tables]{0},
Columns = Table.FromRecords(DataTable[columns]),
ColumnsWithType = Table.Join(Columns, {"type"}, TypeMap , {"AnalyticsTypes"}),
Rows = Table.FromRows(DataTable[rows], Columns[name]), 
Table = Table.TransformColumnTypes(Rows, Table.ToList(ColumnsWithType, (c) => { c{0}, c{3}}))
in
Table
