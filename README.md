# Recombee SHA1-HMAC Postman Pre-request Script 
A simple Pre-request Script for Postman integration to make API requests against the Recombee(recombee.com) AI Recommender Engine. 
It uses the SHA1-HMAC signing method and hashes the timestamp and some path variables into a SHA1 Token. 
Follow the instructions in the script and your good to go. :)

```javascript
/* Pre-requisite
==================
1) Create an Environment (if you don't already have on) and enable it for your request
2) Add two variables to your environment:
    "RecombeeDB" and fill in our DB Name from Recombee as Initial Value
    "PrivateToken" and fill in our Private Token from Recombee as Initial Value
3) Add those new params for your request: 
    "hmac_timestamp" with the value "{{hmac_timestamp}}"
    "hmac_sign" with the value "{{hmac_sign}}"  
    These params should always be at the end of the request to get the script working properly.
    "filter" with the value "{{filter}}"
    "booster" with the value "{{booster}}"
4) Add the following Pre-request Script that computes the two variables and adds them into your environment
5) Fill your and SECRET_KEY with your private key from Recombee(from the corresponding Recombee DB)
6) Your request GET URL should look like this(Example to list all items from Recombee):
    https://rapi.recombee.com/{{RecombeeDB}}/items/list/?hmac_timestamp={{hmac_timestamp}}&hmac_sign={{hmac_sign}}
*/

//If you want to use the filter or boosting attributes, you need to set them here because of escaping logics of recombee and Postman. 
//If you don´t want to use them in your request, let them empty.
var filter  = "'active'";
var booster = "if 'itemId' in \"PROD_42266\" then 10 else 1";

function getParams(params) {
    
    const encodedParams = encodeURIComponent(params)
        .replace(/{/g, "%7B")
        .replace(/}/g, "%7D")
        .replace(/'/g, "%27");

    return encodedParams;
}

function getPath(url) {
    var pathRegex = /.+?\:\/\/.+?(\/.+?)(?:#|\&hmac|\?hmac|$)/;
    var result = url.match(pathRegex);
    return pm.variables.replaceIn(result) && result.length > 1 ? result[1] : ''; 
}
 
function getToken(requestUrl) {
    var SECRET_KEY = pm.environment.get("PrivateToken");

    var requestPath = getPath(requestUrl);

    var querystringregex = /\?/;
    var querystringexist = requestPath.match(querystringregex);
    if(querystringexist != null && querystringexist.length > 0) {
        var separator = "&";
    }
    else {
        var separator = "?";
    }
    
    var PathWithTimestamp = requestPath + separator + "hmac_timestamp=" + getTimestamp();
    var SignedPath = CryptoJS.HmacSHA1(PathWithTimestamp, SECRET_KEY)

    return SignedPath;
}

function getTimestamp() {
    var timestamp = Math.floor(Date.now() / 1000);
    return timestamp;
}

if(booster == null || booster == "") {
    pm.request.url.query.remove("booster");
}
if(filter == null || filter == "") {
    pm.request.url.query.remove("filter");
}

postman.setEnvironmentVariable("filter", getParams(filter));
postman.setEnvironmentVariable("booster", getParams(booster));
postman.setEnvironmentVariable('hmac_timestamp', getTimestamp());
postman.setEnvironmentVariable('hmac_sign', getToken(pm.request.url.getRaw()));
```
