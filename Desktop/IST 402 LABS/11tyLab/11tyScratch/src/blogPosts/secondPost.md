---
layout: post.njk 

title: This is my second dev.to post titled Feeding IP addresses into the API of GeoIP

---
## Feeding IP addresses into the API of GeoIP to Obtain Geographic Coordinates (Lat/Long) 

This will be a brief synopsis and run through on how latitude and longitude coordinates can be detected via GeoIP. Particularly, by feeding IP addresses into the GeoIP API. You can follow along with my [github repo](https://github.com/RajivThummala-psu/ip-project/blob/master/src/LocationFromIP.js). 

Before we get started, let's import the necessary dependencies within your project. Since we are going to be wiring up the data to a wikipedia-query tag, we will need to import the following dependency: @lrnwebcomponents/wikipedia-query/wikipedia-query.js

_Make sure you are using the right syntax_

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r8ycv2suy22w095bxten.png)

To complete the setup process we will also need to do npm install @lrnwebcomponents/wikipedia-query

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/csmpiw6tqrlsky52cf0y.png)
 
As you parse through the constructor repo, you will notice that the first API you run into is: **https://freegeoip.app/json/**

This is the GeoIP API. Unfortunately, at the time of this writing, the server is down so I am unable to provide an overview of all the services this API offers in particular. However, you can head over to this link to poke around. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vyob9fdinxuv9lscbl5k.png)

**Deciphering the GeoIP API?**
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s433cvg4apylswqwr78k.png)
 
You will notice that the geoip is set to the locationEndpoint. 

An endpoint, as adumbrated by [smartbear.com](https://smartbear.com/learn/performance-monitoring/api-endpoints/#:~:text=Each%20endpoint%20is%20the%20location,to%20carry%20out%20their%20function.&text=The%20place%20that%20APIs%20send,lives%2C%20is%20called%20an%20endpoint.), enables  

> APIs to transfer vital information, processes, transactions, and more. API usage will only increase as time goes on, and making sure that each touchpoint in API communication is intact is vital to the success of each API. Endpoints specify where resources can be accessed by APIs and play a key role in guaranteeing the correct functioning of the software that interacts with it.  In short, API performance relies on its ability to communicate effectively with API Endpoints.

Thus, this will serve as an endpoint in which the GeoAPI will retrieve information regarding the client's location to carry out an operation. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fcs1fkmabm1x55j3lrhg.png)

You will also notice that GeoIP is referenced in the async getGeoIPData() method. Let's take some time to pick apart what is going on here.

`async getGEOIPData() {
    const IPClass = new UserIP();
    const userIPData = await IPClass.updateUserIP();
    return fetch(this.locationEndpoint + userIPData.ip)
      .then(resp => {
        if (resp.ok) {
          return resp.json();
        }
        return false;
      })
      .then(data => {

        console.log(data);
        this.state = data.region_name; //avoided making the same mistake as last time (reminder --> region name)
        this.city = data.city;
        this.location = `${data.city}, ${data.state}`;
        this.lat = data.latitude; 
        this.long = data.longitude;

        console.log(`Your coordinates: ${this.lat} ${this.long}`);
        return data;
      });`

To make it easier, lets split this up into 2 parts to get a better understanding: Obtain information from the API & Feed into LitElement based web component to provide stateful updating of the DOM

**Obtain information from the API**

Lets begin by taking a look at this segment:

`async getGEOIPData() {
    const IPClass = new UserIP();
    const userIPData = await IPClass.updateUserIP();
    return fetch(this.locationEndpoint + userIPData.ip)
      .then(resp => {
        if (resp.ok) {
          return resp.json();
        }
        return false;
      })`

The fetch command in particular should strike you as new if you have not worked with this language much. It was for me atleast. However, you will quickly learn from a google search that it works similar to the throw and catch commands from java. 

What is happening here is that the program is essentially throwing out a mandatory request for the program to provide information regarding the location endpoint, which we instantiated in the constructor. If the program does not get a response, it will catch it (through returning false in this case). As stated earlier, the location endpoint serves as a checkpoint for the API to retrieve information regarding the client's location. 

What you will notice also is that "userIPData.ip" is also being fetched. Lets take a look at this further. You can follow along in my repo if that is easier!

`async updateUserIP() {
    return fetch(this.ipLookUp)
      .then(resp => {
        if (resp.ok) {
          return resp.json();
        }
        return false;
      })
      .then(data => {
        this.ip = data.ip;
        this.cityYouAreIn = data.city;
        this.countryYouAreIn = data.country;

        this.location = `${data.city}, ${data.country}`;

        // this.location = "Your location is: " + data.cityYouAreIn+ ", " + data.countryYouAreIn; //understood that the data. references the getproperties method
        return data;
      });
  }`

We will notice here that the updateUserIP method is being used to retrieve information regarding the user's ip address and listing readable info regarding their whereabouts (city & country). It is also fetching ipLookUP (https://ip-fast.com/api/ip/?format=json&location=True), which can be found in the constructor in this file. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/oa1uuy43jdvn5qx14amy.png)

ipLookup references an API called ip-fast. ip-fast is an API that analyzes the client's network activity to approximate their location. In case of error, it will return false to catch. 

Both of these API's are thus added together to obtain the geoIPdata. Therefore, what we essentially have here in the async getGEOIPData() method enables us to leverage 2 APIs to retrieve the information that we need. But what do we do now that the data has been fetched?

**Feed into LitElement based web component to provide stateful updating of the DOM**

`  })
      .then(data => {

        this.state = data.region_name; //avoided making the same mistake as last time (reminder --> region name)
        this.city = data.city;
        this.location = `${data.city}, ${data.state}`;
        this.lat = data.latitude; 
        this.long = data.longitude;

        console.log(`Your coordinates: ${this.lat} ${this.long}`);
        return data;
      });`

Now if we take a look at this second half of the method, we will notice that the data is actually being entered into the console for use. When I first ran into this section of the program, I actually found it quite easy to interpret despite being unfamiliar with this syntax. 

We start of with "then the data"...

Is used to retrieve information regarding the client's state. You must call for state through region_name as opposed to just data.state (don't make the same mistake I did). Next you can see that the city and location are retrieved. We also add both latitude and longitude into our instances so that we can obtain the geographical coordinates. Ensure that you are doing data.latitude and data.longitude NOT data.lat and data.long. 

These instances are then simply fed into the console.

`console.log(`Your coordinates: ${this.lat} ${this.long}`);
        return data;`

From here, all you have to do is render. I referenced [btopro's codepen](https://codepen.io/btopro/pen/yLNmVbw) to do this, as you can see in my repo. 

After copious efforts getting all the bugs out of my code, I am close to getting an accurate solution. I will continue to update the repo as I optimize the program.

As you can see, leveraging public API's to perform functions are seamless and potent. There are a multitude of micro services across the web that can be utilized to achieve a goal in a non-taxing manner. 

NASA Rendering in particular is a great one, which provides the ability to render images from NASA by leveraging their [public api](https://api.nasa.gov/api.html#apod). This enables individuals that are not affiliated with NASA to easily access this large stack of information with ease. 

As highlighted by [wilsjame](https://wilsjame.github.io/how-to-nasa/), these are just some of the [many APIs](https://api.nasa.gov/api.html#apod) that can be used: 

- Near Earth Object Web Service (NeoWs): Access to near Earth asteroid information.

- Earth Polychromatic Imaging Camera (EPIC): Full disc imagery of the Earth.

- Earth Observatory Natural Event Tracker (EONET): Prototype web service providing continuously updated natural event metadata, such as storm imagery, gathered from the Earth’s surface.
NASA Image and Video Library: Access to the NASA Image and Video Library.

- Sounds (beta): Access to space sounds via SoundClound with some of the hassle abstracted away.

- NASA rendering; render images from NASA using their public API to render the data


Good luck on your attempt and I thank you for taking your time to read this post. 






