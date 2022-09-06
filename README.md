for this project we need a third party API for Retrieving IP Address Geolocation

RapidAPI’s API marketplace offers a rich collection of APIs suited for different applications. From industry verticals such as finance, travel, and healthcare, to technology domains such as data analytics, visual recognition, and cloud communication, RapidAPI hosts thousands of APIs from different API providers to suit your needs.

For our demo REST API interface, we can go with the IP Geolocation API. This API is simple to use. It is a single endpoint that returns geo location-related information for a public IP address. 

Before leveraging this API, you have to signup for a free RapidAPI account and subscribe to this API. Follow along with the steps below.

Step 1. Sign Up for RapidAPI Account
Sign up for a free RapidAPI developer account. With this account, you get a universal API Key to access all APIs hosted in RapidAPI’s API marketplace.

RapidAPI is the world’s largest API marketplace, with over 10,000 APIs and a community of over 1,000,000 developers. Our goal is to help developers find and connect to APIs to help them build amazing apps.

Step 2. Subscribe to IP Geolocation API
Once signed in, log in to your RapidAPI account and access the API console. 
Implementing the IPMon REST API Interface with Node.js
Let’s christen this demo API as IPMon API. With the IPMon API interface finalized earlier and the backend API chosen through RapidAPI, you are now ready to take the plunge into coding. 

Before you start writing the code, make sure that you have the right programming environment to develop and test the API locally.

Prerequisites
Node.js Runtime: Download and install Node.js from https://nodejs.org/en/. 
cURL: Download and install the curl command. This tool will be used for testing the API interface.  


IDevelopment Steps

Launch your favorite code editor and open a command-line terminal. Follow the steps below to complete the development for the IPMon API service.

Step 1: Create a Node.js project
On the terminal, run the following command to create the top-level directory for this project under your chosen path.

mkdir node-api
node-api is your top-level directory, which will have all the project-specific files.  Change to this directory and create a new Node.js project.

cd node-api
npm init -y

The npm init command will initialize an empty project, and generate a file named ‘package.json’.  We are not following the Node.js toolchain for this application, so you can ignore this file’s contents. 

Create a new file named ‘server.js’. 

touch server.js




const http = require ('http'); // to create server
const url = require('url'); // to parse url
const https = require('https');// to send https requests 
 
var ip_table = new Map();// for simplicity using has map to store ip details instead of db

// create basic server and implement handling different requests
const app = http.createServer( async (req, res) => {
    // parse the incoming url
    const parsedURL = url.parse(req.url, true);
    
    //check for  POST /api/ipmon/ip
    if ( parsedURL.pathname === '/api/ipmon/ip' && req.method === 'POST') {
        // check if ip is prsent in query else send error
        if(!parsedURL.query.ip) {
            res.statusCode = 400;
            res.end("IP address is required, please add IP address in the query");
        } else {
            handleCreate(parsedURL.query.ip,res);
        }
        
    }
    //check for GET /api/ipmon/ip/show 
    else if (parsedURL.pathname === '/api/ipmon/ip/show' && req.method === 'GET'){
        handleShow(res);              
    }
    //check for  PUT /api/ipmon/ip/{ip_address}
    else if (parsedURL.pathname.startsWith('/api/ipmon/ip') && req.method === 'PUT'){
        
        var ipAddr = extractIPAddress(parsedURL.pathname)
        
        if(!ipAddr.length) {
            res.statusCode = 400;
            res.end("Invalid IP Address");
        } else {
            handleUpdate(ipAddr,res);
        }
    }
    //check for  DELETE /api/ipmon/ip/{ip_address}
    else if (parsedURL.pathname.startsWith('/api/ipmon/ip') && req.method === 'DELETE'){
        
        var ipAddr = extractIPAddress(parsedURL.pathname)
        
        if(!ipAddr.length) {
          
            res.statusCode = 400;
            res.end("Invalid IP Address");
        } else {
            handleDelete(ipAddr,res);
        }
    }
    //check for GET /api/ipmon/ip/{ip_address}
    else if (parsedURL.pathname.startsWith('/api/ipmon/ip') && req.method === 'GET'){
        var ipAddr = extractIPAddress(parsedURL.pathname)
        
        if(!ipAddr.length) {
            res.statusCode = 400;
            res.end("Invalid IP Address");
        } else {
            handleRead(ipAddr,res);
        }
    } 
    // if url doent match any send error
    else {
        res.statusCode = 400;
        res.end("API Endpoint Not Supported");
    }
});//End of create server.

app.listen(4000);

/* function to handle create */
function handleCreate( ip,res) {
    // call geolocation api and get the details
    getGeolocation( ip ).then( response => {
        // update the database - map in this case
        updateTable(response);
        // set the header and status code success and return the details of the ip
        res.setHeader('content-type', 'Application/json');
        res.statusCode = 200;
        res.end(JSON.stringify(ip_table.get(ip)));
 
    },
    error => {
        res.statusCode = 400;
        res.end(error);
    } )
 
}//End of handle create
//function to start handleshow
function handleShow(res){
    // set the header and status
    res.setHeader('content-type', 'Application/json');
    res.statusCode = 200;
    // create an array from the map 
    array = Array.from(ip_table, ([name, value]) => ({ name, value }));
    // send the response array
    res.end(JSON.stringify(array));
}
// function to handle read request
function handleRead(ip,res){
    // check if the ip is present in table
    if(ip_table.has(ip)){
        // set header, status code and send the entry
        res.setHeader('content-type', 'Application/json');
        res.statusCode = 200;
        res.end(JSON.stringify(ip_table.get(ip)));
    } else {
        // ip not found send error
        res.statusCode = 404;
        res.end("Read: IP "+ip+" not found");
    }
 
}
// function to handle delete request
function handleDelete(ip,res){
    // check if ip is in the table
    if(ip_table.has(ip)){
        // delete and send the success response
        ip_table.delete(ip);
        res.statusCode = 200;
        res.end("Delete:  "+ip+" deleted");
    } else {
        // ip not found send error
        res.statusCode = 404;
        res.end("Delete: IP  "+ip+" not found");
    }
 
}
// function to handle update request
function handleUpdate(ip,res){
    // check if the ip is present in table
    if(ip_table.has(ip)){
        // update the details by calling geoplocation api
        getGeolocation( ip ).then( response => {
            // set header, status code, update table and send response
            res.setHeader('content-type', 'Application/json');
            res.statusCode = 200;
            updateTable(response);
            res.end(JSON.stringify(ip_table.get(ip)));
 
        },
        error => {
            // send error 
            res.statusCode = 400;
            res.end(error);
        } );
    } else {
        // ip not found in table so send error 
        res.statusCode = 404;
        res.end("Update: IP "+ip+" not tracked")
    }   
}
const rapidAPIBaseUrl = "https://rapidapi.p.rapidapi.com/json/?ip=";

function getGeolocation( ipAddress ) {
            // initilize http.rquest object
            var req = https.request;
            // initialize header with the required information to call geolocation api. 
            var header = {
                            "x-rapidapi-host": "ip-geolocation-ipwhois-io.p.rapidapi.com",
                            "x-rapidapi-key": "4e0ac667ebmsh0a6ed0aad4a68b3p1a49c0jsn700bec2a6c0e",
                            "useQueryString": true
                        };
            // add the query string including the IP address
            var query_string = { "ip" : ipAddress };
            // set the options parameter
            var options = {
                                headers: header,
                                query: query_string
                            };
            // form the url
            const url =  rapidAPIBaseUrl + ipAddress ;
 
            return new Promise ( ( resolve, reject) => {
                                
                    https.get( url, options, res  => {
                       
                        let data = "";
                         //data is received in chunks, so uppend data as and when received
                        res.on( 'data', function(response) {
                                    data = data + response;
                        });
                        // handle error if any
                        res.on( 'error', function(err) {
                            console.log("Error");
                            console.log(err);
                        })
                        // if end of data return the final chunk
                        res.on( 'end', () => {
                            resolve(  JSON.parse(data) ); 
                        });
                    });//Endn of http
            }); //end of return promise
}//End of getGeolocation
//function for update table
function updateTable(entry){
    // get current date to update last update the time
    var time = new Date();
    // add the entry to table ip is the key and country, city and last updated time are stored
    ip_table.set(entry.ip, {"ip":entry.ip,"country":entry.country,"city":entry.city,"lastUpdated":time});
 
}
// function for extract ip address
function extractIPAddress(path){
  
    var ipAddress = path.substring(path.lastIndexOf("/")+1,path.length);
  
    if (/^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/.test(ipAddress))
      {
        return ipAddress;
      }
      else
      {
        console.log("invalid IP Address")
        return "";
      }
  
  } // End of extractIPAddress
