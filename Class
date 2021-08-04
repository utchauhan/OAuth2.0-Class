public class 3rdutil {
    public static String 3rdAppQuerry(){
        String query = SOQLUtil.buildSOQLQuery('object__c',filter_name,filter);
        //SOQLUtil class code in SOQL util repo
        return query;
    }

    public static PageReference doAuthorizationQuickBooks(){

        String authorization_endpoint = 'https://appcenter.intuit.com/connect/oauth2';

        String scope = 'com.intuit.quickbooks.accounting';

        String final_EndPoint = authorization_endpoint+'?client_id='+client_Id+'&response_type=code&scope='
        +scope+'&state=123445633443&redirect_uri='+redirect_URI;

        PageReference pageRef = new PageReference(final_EndPoint);
        return pageRef;
    }

    public PageReference accesstoken(){
        String query = 3rdAppQuerry();
        List<object__c> result = Database.query(query);
        object__c newobj = new object__c;
        if(result != null && !result.isEmpty()){
            newobj = result.get(0);
        }
        else {
            return null;
        }
        
        String code = Apexpages.currentPage().getParameter().get('code');
        String realmID = Apexpages.currentPage().getParameter().get('code');
        String baseurl = System.URL.getSalesforceBaseUrl().toExternalForm()+'/apex/'+newobj.pagename__c;
        //pagename__c is the name of apex vfpage that api is redirected to
        String requestbody = 'grant_type=authorization_code&code='+code+'&redirect_uri'+baseurl;
        String errorMessage = '';

        String endpoint = newobj.tokenurl__c;
        String basic = newobj.ClientId__c+':'+newobj.Clientsecret__c;
        String encodedString = 'Basic '+EncodingUtil.base64Encode(blob.Valueof(basic));

        Http http = new Http();
        HttpRequest req = new HttpRequest();
        req.setHeader('Content-Type', 'application/json');
        req.setHeader('Accept', 'application/json');
        req.setHeader('Authorization', encodedString);
        req.setMethod('POST');
        req.setEndpoint(endpoint);
        
        try
        {
            HTTPResponse response = http.send(req);
            if(response.getStatusCode() == 200 || response.getStatusCode() == 201)
            {
                System.debug();
                system.debug(System.LoggingLevel.DEBUG, 'Body '+response.getBody());

                JSONWrapper responseWrapper = JSONWrapper.parse(response.getBody());

                map<String,object> tokenmap = new map<String,object>();
                tokenmap.put('realmid__c',realmid);
                tokenmap.put('refresh_token__C',token.refresh_token);
                tokenmap.put('access_token__c',token.access_token);
                tokenmap.put('expires_in__C',decimal.valueof(token.expires_in));
                tokenmap.put('token_expires_in__c',system.now().addseconds(token.token_expires_in));
                
                //update custom metadata tokenmap -> go to Metadata repo for code
                
                Apexpages.addmessage(new Apexpages.message(Apexpages.severity.CONFIRM,'Succesfully Authenticated! You can close the win'));       
            }
            else{
                errorMessage = 'Unexpected Error while communicating with API. '+ 'Status '+ response.getStatus() +'and Status Code'+response.getStatusCode();
                Apexpages.addmessage(new Apexpages.message(Apexpages.severity.ERROR,response.getBody())); 
            }
        }
        
        catch(System.Exception e)
        {
            if(String.Valueof(e.getMessage()).startsWith('Unauthorized Endpoint')){
                errorMessage = 'Unauthorized endpoint: An Administrator must go to setup -> Administer -> Security Control -> Remote Site Settings and add '
                + endpoint +' Endpoint';           
            }
            else{
                errorMessage = 'Unexpected Error while communicating with XYZ. '+ 'Status '+ response.getStatus() +'and Status Code'+response.getStatusCode();
            }
            system.debug(System.LoggingLevel.DEBUG, 'Exception Executed '+errorMessage);
        }

    }

    public static Map<String,object> refreshToken(object__c newobj){   
        //object__c is the custom data created to store Oauth creds

        String errorMessage = '';
        String endpoint = newobj.tokenurl__c;
        String basic = newobj.ClientId__c+':'+newobj.Clientsecret__c;
        String encodedString = 'Basic '+EncodingUtil.base64Encode(blob.Valueof(basic));
        String baseurl = System.URL.getSalesforceBaseUrl().toExternalForm()+'/apex/'+newobj.pagename__c;

        String requestbody = 'grant_type=refresh_token&refresh_token='+newobj.refresh_token__C;

        Http http = new Http();
        HttpRequest req = new HttpRequest();
        req.setHeader('Content-Type', 'application/json');
        req.setHeader('Accept', 'application/json');
        req.setHeader('Authorization', encodedString);
        req.setMethod('POST');
        req.setEndpoint(endpoint);

        try
        {
            HTTPResponse response = http.send(req);
            if(response.getStatusCode() == 200|| response.getStatusCode() == 201)
            {
                System.debug();
                system.debug(System.LoggingLevel.DEBUG, 'Body '+response.getBody());

                JSONWrapper responseWrapper = JSONWrapper.parse(response.getBody());

                map<String,object> tokenmap = new map<String,object>();
                tokenmap.put('realmid__c',realmid);
                tokenmap.put('refresh_token__C',token.refresh_token);
                tokenmap.put('access_token__c',token.access_token);
                tokenmap.put('expires_in__C',decimal.valueof(token.expires_in));
                tokenmap.put('token_expires_in__c',system.now().addseconds(token.token_expires_in));
                //update custom metadata go to Metadata repo for code     
            }
            else{
                errorMessage = 'Unexpected Error while communicating with API. '+ 'Status '+ response.getStatus() +'and Status Code'+response.getStatusCode(); 
            }
        }
        catch(System.Exception e)
        {
            if(String.Valueof(e.getMessage()).startsWith('Unauthorized Endpoint')){
                errorMessage = 'Unauthorized endpoint: An Administrator must go to setup -> Administer -> Security Control -> Remote Site Settings and add '
                + endpoint +' Endpoint';           
            }
            else{
                errorMessage = 'Unexpected Error while communicating with XYZ. '+ 'Status '+ response.getStatus() +'and Status Code'+response.getStatusCode();
            }
            system.debug(System.LoggingLevel.DEBUG, 'Exception Executed '+errorMessage);
        }
        return tokenmap;

    }

    public static Boolean check_token_validity(object__c newobj){
        Boolean isvalid = True;
        if(newobj.token_expires_in__c < System.now()){
            isvalid = False;
        }
        return isvalid;
    }

    public static void api_operation(apiresponsebodyclass xyz){
        object__c newobj = creds();
        String accesstoken = newobj.access_token__c;
        Boolean isvalid = check_token_validity(newobj);
        Map<string, object> refresh_token_Map = new Map<string, object>();
        if(!isvalid){
            refresh_token_Map = refreshToken(newobj);
            accesstoken = (string)refresh_token_Map.get('access_token__c');
        }
        String endpoint = newobj.environment == 'Sandbox'? newobj.sandbox_baseUrl__c :  newobj.production_baseUrl__c;
        
        String customurl = ''; //based on the api requirement 
        customurl = customurl.replace('{realmid}',newobj.realmid__c);
        
        String finalendpoint = endpoint+customurl//plus any other parameters
        
        string errorMessage = '';
        String requestbody = ''; 
        // or after populating dezired values for xyz instance of API_response_body_class
        // xyz.fieldvalues = blah blah; 
        // perform String requestbody = System.JSON.serealize(xyz)
        Http http = new Http();
        HttpRequest req = PrepareRequest.prepareRequest(finalendpoint,accesstoken,'POST',requestbody);
        HTTPResponse response = new HTTPResponse();
        try
        {
            response = http.send(req);
            if(response.getStatusCode() == 200|| response.getStatusCode() == 201)
            {
                System.debug();
                system.debug(System.LoggingLevel.DEBUG, 'Body '+response.getBody());

                API_response_body_class responseWrapper = API_response_body_class.parse(response.getBody());
// use List<API_response_body_class> responses = (List<API_response_body_class>)System.JSON.deserialize(response.getBody(),List<API_response_body_class>.class);
// when dealing with multiple responses

            }
            else{
                errorMessage = 'Unexpected Error while communicating with API. '+ 'Status '+ response.getStatus() +'and Status Code'+response.getStatusCode(); 
            }
        }
    }
}

public class JSONWrapper 
{
    public String refresh_token;
    public String access_token;
    public String token_type;
    public String expires_in;
    public String token_expires_in;

    public static JSONWrapper parse(String body) 
    {
        return (JSONWrapper)System.JSON.deserialize(body, JSONWrapper.class);
    }
}

global without sharing class PrepareRequest{
    public static HttpRequest prepareRequest(String endpoint, String accessToken, String method, String requestBody){
        HttpRequest req = new HttpRequest();
        req.setMethod(method);
        req.setEndpoint(endpoint);
        req.setHeader('Content-Type', 'application/json');
        req.setHeader('Accept', 'application/json');
        req.setHeader('Authorization', 'Bearer '+accessToken);
        if(!String.isblank(requestBody)){
            req.setBody(requestBody);
        }
        return req;
    }
}





