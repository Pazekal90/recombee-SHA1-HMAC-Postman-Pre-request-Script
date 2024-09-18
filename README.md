# Recombee SHA1-HMAC Postman Pre-request Script 
A simple Pre-request Script for Postman integration to make API requests against the [Recombee AI](recombee.com) Recommender Engine. 

It uses the SHA1-HMAC signing method and hashes the timestamp and some path variables into a SHA1 Token.

Follow the instructions in the script and your good to go. :)

```javascript
/* Pre-requisite
==================
1) Create an Environment (if you don't already have on) and enable it for your request
2) Add two variables to your environment:
    * "RecombeeDB" and fill in our DB Name from Recombee as Initial Value
    * "PrivateToken" and fill in our Private Token from Recombee as Initial Value
    * "BaseUrl" and fill in API endpoint (e.g. https://rapi.recombee.com)
3) Add params for your request:
    * "filter" with the value "{{filter}}"
    * "booster" with the value "{{booster}}"
4) Add the following Pre-request Script that computes the two variables and adds them into your environment
5) Your request GET URL should look like this (Example to list all items from Recombee):
    `{{BaseUrl}}/{{RecombeeDB}}/items/list/`

Example URLs:
* Recommend items to item:
    `{{BaseUrl}}/{{RecombeeDB}}/recomms/items/{{ITEM_ID}}/items/`
* Recommend items to item with filter and booster:
    `{{BaseUrl}}/{{RecombeeDB}}/recomms/items/{{ITEM_ID}}/items/?filter={{filter}}&booster={{booster}}`

*/

//If you want to use the filter or boosting attributes, you need to set them here because of escaping logics of recombee and Postman. 
//If you don´t want to use them in your request, let them empty.
cryptojs = require('crypto-js');

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
    var url_replaced = pm.variables.replaceIn(url)
    var pathRegex = /.+?\:\/\/.+?(\/.+?)(?:#|$)/;
    var result = url_replaced.match(pathRegex);
    return result.length > 1 ? result[1] : ''; 
}
 
function getToken(requestUrl, timestamp) {
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
    
    const PathWithTimestamp = requestPath + separator + "hmac_timestamp=" + timestamp;
    const hmac = cryptojs.algo.HMAC.create(cryptojs.algo.SHA1, SECRET_KEY);
    hmac.update(PathWithTimestamp);
    const SignedPath = hmac.finalize().toString();

    return SignedPath;
}

function signUrl() {
    var timestamp = getTimestamp();
    var hmacSign = getToken(pm.request.url.toString(), timestamp);
    
    pm.environment.set('hmac_timestamp', timestamp);
    pm.environment.set('hmac_sign', hmacSign);
    pm.request.url.query.add("hmac_timestamp={{hmac_timestamp}}");
    pm.request.url.query.add("hmac_sign={{hmac_sign}}");
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

pm.environment.set("filter", getParams(filter));
pm.environment.set("booster", getParams(booster));
signUrl();
```
