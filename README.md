# node-weather-app
•	Async Basics
•	Call Stack & Event Loop
•	Callback Functions & APIs
•	Pretty Printing Objects
•	What's Makes up an HTTP Request?
•	Encoding User Input
•	Callback Errors
•	Abstracting Callbacks
•	Wiring Up Weather Search
•	Chaining Callbacks Together
•	Intro to ES6 Promises
•	Advanced Promises
•	Weather App with Promises

Asynchronous Node.js - Weather App
    -Using Google Geolocation API and DarkSky API to get realworld weather information
    -Callbacks, promises, HTTP requests, and more

    Asynch Basics
        -   setTimeout(function(){}, timeoutMilliseconds);

    Call Stack & Event Loop
        - Why does second timeout with 0 delay not happen immed iately?
        - Call Stack keeps track of execution
            - Example 1 - Sync
                - Can add something on top of it
                - Can remove top item
                - Can't remove middle items
                - Starts with main, adds other functions on top.
                - In synchronous code, functions go on and come off one at a time
            - Example 2 - Sync
                - Function definitions get put on the call stack
                - When you call a function, it goes on top of the call stack and executes on top of itself
            Example 3 - Async (async-basics.js)
                - console.log is fine, executes and is off the call stack
                - setTimeout gets registered in NodeAPI, removed from call stack.
                - setTimeout with 0 finishes immediately, gets added to Callback Queue, where functions go when they're ready for callback
                - Callback Queue waits for call stack to empty before firing callback functions
                - First callback function executes before others.
                - Event Loop looks at Call Stack to see if there's anything in it.
                    - If the Call Stack is empty, it runs Callback Queue
                    - If the Call Stack isn't empty, it holds
                - console.log runs, empties call stack with end of main.
                - setTimeout 0 callback runs!
                - setTimeout 2 finishes, goes to Callback Queue, is recognized by Event Loop, and gets added to empty Call Stack!

    Callback Functions & APIs
        - Callback Functions
            - Function that gets passed as an arg to another function that gets called after an event
        - Going to use https://maps.googleapis.com/maps/api/geocode/json link with query strings (?address=${encodedAddress}&key=[API key]) to get a JSON filled with neat things that we want
        - We'll use the request module
            - Create package.json with npm init
            - npm install request --save
            -   request({
                    url: `https://maps.googleapis.com/maps/api/geocode/json?address=${encodedAddress}&key=[API key]`;
                    json: true
                }, (error, response, body) => {
                    console.log(body);
                })
                - request is the URL we're hitting up
                - json: true returns a JSON, saving us a step
                - callback method takes default args error, response, body

    Pretty Printing Objects
        - JS clips objects, not the entire thing.
        - Use JSON.stringify(JSON, spaces)
            - JSON is the object to stringify
            - filter we don't use, usually useless - can leave as undefined
            - spaces is number of spaces to indent
            - Includes all properties!

    What makes up an HTTP Request?
        - HyperText Transfer Protocol
        - Body
            - Data that comes back in the HTTP request
            - Usually the data that gets rendered on your screen
            - Can be JSON information
            - Core data
        - Response
            - Includes StatusCode
            - Body (repeated, is part of the response)
            - Headers - Key/Value pairs
            - Request - Stores info about request that was made
            - Our Headers - Ours says to accept header info back
        - Error
            - Would be an error in making the HTTP request
            - If URL is wrong or machine doesn't have access to the internet
            - Mostly looking at code.
            - Returns 'null' when empty

    Encoding User Input
        - We want to add address as an option
            -   npm install yargs --save //install yargs
            -   const yargs = require('yargs'); // use yargs

                const argv = yargs
                    .options({      //configure option
                        address: {
                            demand: true,
                            alias: 'a',
                            describe: 'Address to fetch weather for',
                            string: true    // Require string data type
                        }
                    })
                    .help() // adds help flag
                    .alias('help', 'h') // alias for help flag
                    .argv;

                console.log(argv);
        - We want to encode the user input
            -   encodeURIComponent(string)
                - Encodes string to be URI compatible
            -   decodeURIComponent(string)
                - Decodes string to be normal
            - Can change URL in request
                -   url: `https://maps.googleapis.com/maps/api/geocode/json?address=${encodeURIComponent(argv.a)}`

    Callback Errors
        - Current app doesn't deal with errors at all
        - We'll add control flow to callback.
            -   if (error) {
                    console.log('Unable to connect to Google serviers.');
                } else if (body.status === 'ZERO_RESULTS') {
                    console.log('Unable to find that address.');
                } else if (body.status === 'OK') {
                    console.log(`Address: ${body.results[0].formatted_address}`);
                    console.log(`Latitude: ${body.results[0].geometry.location.lat}`);
                    console.log(`Longitude: ${body.results[0].geometry.location.lng}`);
                }

    Abstracting Callbacks
        - Moving the complex call to request to another file
        - Changed request call to geocode function, modularized to use callback

    Wiring Up Weather Search
        - Signed up for DarkSky API to make API calls
        - Made new request in app.js with API key and static lat/lng to get current temperature
            -used API key: 9bc212d24b0b7a46f48f2ef046b743ce
        - Added error handling on the request

    Chaining Callbacks Together
        - Break out weather request call to another file
        - Take weather and make like geocode
            - make lat/lng args
            - Replace console.log in error handling with callback
        - Put weather.getWeather() call inside of the geocode.geocodeAddress() callback function

    Intro to ES6 Promises
        - Aim to solve problems that arise with async code
        - Can use promises to replace callbacks - sort of?
        - promise.js
            -   var somePromise = new Promise(function(resolve, reject) {

                }); 
                - creates a new promise
                - If promise is fulfilled, it is resolved
                - If promise fails, it is rejected
            -   somePromise.then(function(arg){
                    console.log('Success:', message);
                })
                - Gets called when promise succeeds with arg from resolve
            -   somePromise.then(function, function() ={

                })
                - Second function handles failed promises
            - Promises only get to call .resolve() or reject() once.
            - Until it resolves or rejects, it is pending.

    Advanced Promises
        - Can wrap a function in a promise to make it promise-compatible, giving then functionality

        - Promise chaining
            - return a new Promise inside then.
            - Chain another .then() after current .then()
            - Can remove reject methods from multiple .then() functions, include one .catch() method and make that the error response.

    Weather App With Promises
        - Going to look at Axios library, which works like request but with promises
        - Going to recreate weather app
        -   var encodedAddress = encodeURIComponent(argv.address);
            var geocodeURL = `https://maps.googleapis.com/maps/api/geocode/json?address=${encodedAddress}`;

            axios.get(geocodeURL).then((response) => {
                if (response.data.status === 'ZERO_RESULTS') {
                    throw new Error('Unable to find that address.');
                }
                
                var lat = response.data.results[0].geometry.location.lat;
                var lng = response.data.results[0].geometry.location.lng;;
                var weatherURL = `https://api.darksky.net/forecast/d6d5b17fed6ea84ad5109d124670b7aa/${lat},${lng}`,
                console.log(response.data.results[0].formatted_address);
                return axios.get(weatherURL);
            }).then((response) => {
                var temperature = response.data.currently.temperature;
                var apparentTemperature = response.data.currently.apparentTemperature;
                console.log(`It's currently ${temperature}. It feels like ${apparentTemperature}.`)
            }).catch((e) => {
                if (e.code === 'ENOTFOUND') {
                    console.log('Unable to connect to API servers.');
                } else {
                    console.log(e.message);
                }
            });

    Extra Features (for Weather App)
        - Load in additional information
        - Default location ability.
