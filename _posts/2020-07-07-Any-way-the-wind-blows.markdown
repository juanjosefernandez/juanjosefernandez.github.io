---
title:  "Weather-Or-Not: Web App Process"
author: Juan Jose Fernandez
comments: true
layout: post
date:   2020-07-07 00:04:51 -0400
categories: programming
tags:
- weather-or-not
- missy moreno
- React
- Javascript
- OpenWeather API
- MapBox API
- api 
- web app
image: /assets/images/weatherornot.gif
vertical: Code
excerpt: Earlier this summer my buddy Missy Moreno and I decided to make inroads in learning how to work with React and APIs. We decided to make a cheerful weather web app that had some kind of suggestion system based on the weather forecast at a given location. 
# excerpt_separator: "<!--more-->"
summary: Earlier this summer my buddy Missy Moreno and I decided to make inroads in learning how to work with React and APIs. We decided to make a cheerful weather web app that had some kind of suggestion system based on the weather forecast at a given location. 
music: <a href="https://www.youtube.com/watch?v=4x0fPZrPV3M">Romeo Void - Never Say Never</a>

---
<style>
.bar{
    height: 10px;
    background: #bc4e9c;  /* fallback for old browsers */
    background: -webkit-linear-gradient(to top, #f80759, #bc4e9c);  /* Chrome 10-25, Safari 5.1-6 */
    background: linear-gradient(to top, #f80759, #bc4e9c); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
    }
</style>
![Animation of the web app in use](/assets/images/weatherornot.gif)

Earlier this summer my buddy [Missy Moreno](https://www.missymoreno.com/) and I decided to make inroads in learning how to work with React and APIs. We decided to make a cheerful weather web app that had some kind of suggestion system based on the weather forecast at a given location. That was the genesis of Weather-Or-Not. Here's a [link to the Github repository](https://github.com/juanjosefernandez/weather-or-not) where we keep the code.
<div class="bar"></div>
Having just recently learned how to write successful code in ReactJS, we dove in and got to work. We had plenty of messy false starts before we landed on a good component structure that allowed for us to do what we wanted. My intended focus was on what learning how to make API calls correctly and successfully using components in ReactJS.

Not having ever used an API on my own, I thought it would be good to start with some guard rails. We used [this tutorial from RapidAPI](https://rapidapi.com/blog/weather-app-react/). It didn't connect to the endpoints that we needed it to, but that was ok, we needed to figure out how this all worked!

After we got that working we had a basic web-app with inroads of our desired core functionality. We took a step back and discussed what we needed to do differently to get where we wanted to go. Notably, we wanted to get the weather conditions for the next 5 days, not just the current weather. This required using a different API endpoint which required latitude and longitude. So, we doctored up our code to hit the appropriate API endpoints and we were in business.

At that point, we were manually entering Pittsburgh's latitude and longitude to test. No good! Naturally that's not a reasonable way to have user's enter their location. Turns out we needed to connect to another API to help us transform city names into latitude and longitude coordinates! 

[Mapbox](https://docs.mapbox.com/api/) was the ticket. We used their [forward geocoding](https://docs.mapbox.com/api/search/#forward-geocoding) feature of the API. At this point we had the intended usage for the application's user interface. 

Because Javascript is a collection of asynchronous processes, it was necessary to chain the calls in such a way where you never call the Openweather API until you've gotten a successful response from the Mapbox API. As you'll see below the two fetches are linked by a callback that you can see in line 29. 

Figuring this out was its own kind of puzzle. I had a lot of fun figuring out how to make this work.  This post on [how to use the fetch() method to make multiple API calls with vanilla JavaScript ](https://gomakethings.com/how-to-use-the-fetch-method-to-make-multiple-api-calls-with-vanilla-javascript/) was immensely helpful.

Here's a look at the function that makes this work!

{% highlight javascript linenos %}
function getWeather(e){
    e.preventDefault();
    console.log("CITY UPON BUTTON PRESS: ", city);
    // Clear state in preparation for new data
    setError(false);

    console.log("in getWeather")
    setResponseObj({});
    setLoading(true);
    setSafe(false);

    var post;

    // Call the mapbox API
    fetch(`https://api.mapbox.com/geocoding/v5/mapbox.places/${city}.json?access_token=MAPBOXACCESSTOKEN`).then(function (response) {
        if (response.ok) {
            setLoading(false);
            return response.json();
        } else {
            return Promise.reject(response);
        }
    }).then(function (mapBoxData) {

        // Store the post data to a variable
        
        // responseObj = data;

        // Call the openweatherAPI
        return fetch(`https://api.openweathermap.org/data/2.5/onecall?lat=${mapBoxData.features[0].center[1]}&lon=${mapBoxData.features[0].center[0]}&exclude=minutely,hourly&units=imperial&appid=MAPBOXAPIID`);

    }).then(function (response) {
        if (response.ok) {
            return response.json();
        } else {
            return Promise.reject(response);
        }
    }).then(function (weatherData) {
        console.log("userData is: ", weatherData);
        responseObj = weatherData;
        setResponseObj(weatherData);

        setSafe(true);
        console.log("STORED AT RESPONSE OBJ AT THIS MOMENT: ", JSON.stringify(responseObj));

    }).catch(function (error) {
        console.warn(error);
    });
    
    return; 
}
{% endhighlight %}

We passed the OpenWeather response object over to the conditions component, which contains the strip components.

    <Conditions
    // this component holds our strips.
    responseObj={responseObj}
    error={error} //new
    loading={loading} //new
    safe={safe}
    />

Each strip was its own component that was passed a copy of responseObj so that it could pull out what it needed from the JSON blob. The strips were able to be handle the data according to the pertaining day via the **day** variable. The **whichStrip** variable controlled how the CSS rendered each strip's color.

    <Strip whichStrip="StripOne" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={0}/> 
    <Strip whichStrip="StripTwo" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={1}/> 
    <Strip whichStrip="StripThree" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={2}/> 
    <Strip whichStrip="StripFour" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={3}/> 
    <Strip whichStrip="StripFive" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={4}/> 

Each strip is constructed as follows. You can see the **whichStrip** prop that is passed into this component dictates what class is used to style the strip.

    <div > 
        <container>
            <section className={props.whichStrip}>
            <p>{props.safe ?  "Lo: " + JSON.stringify(props.responseObj.daily[props.day].temp.min) + "°F" : ""}</p>
            <p>{props.safe ? "Hi: " + JSON.stringify(props.responseObj.daily[props.day].temp.max) + "°F"  : ""}</p>
            <p>{props.safe ?  "Main Description:" + JSON.stringify(props.responseObj.daily[props.day].weather[0].main) : ""}</p>
            <p>{props.safe ?  "Sub Description: " + JSON.stringify(props.responseObj.daily[props.day].weather[0].description) : ""}</p>
            <div>{props.safe ? <img src={"http://openweathermap.org/img/wn/"+ iconId + "@2x.png"}></img> : ""}</div>
            </section>
        </container>
    </div>

It's not the cleanest or the most clever code, but it gets the job done and I want to show my work here! It's a little clunky and could work far more efficiently. But it works! And that's what counts for me and Missy in our journey of knowledge. Missy has been having lots of fun playing with the styling, as you can see in the GIF above. It's wacky and I love it.

Missy has a big vision of what she wants this web app to be in the future. Based on the weather the user gets all kinds of different suggestions to help them have a cheery day. 

☁️***Whether or not the weather is "good", there's lots you can do to have a good day.*** ☁️

The idea is to create a table of weather conditions that dictated different outputs. For example if it is  sunny but the high temperature is less than 35°, it makes sense to suggest going outside as it's likely going to be a cool and crisp day. Likewise, say it is above 80° and OpenWeather says the weather can be described as "heavy rain", it's going to be a muggy day where you'll likely be cooped up inside. Board games with cool drinks are likely a good suggestion. 

>Idea: Build a table of activity suggestions that are offered to the user for suggestions as to what they might think to do in the upcoming days. The idea would be to have a collection of rainy weather activities that are randomly selected. ⛅

This is something that Missy and I need to sit down to do. Another thing we were thinking of was connecting a recipe API that would be called and passed different variables based on the weather conditions. Fish tacos recipes when it's thundering, mimosas when it's hailing, etc... 

As you can see the weather can trigger a lot of different responses. Rather than lose ourselves in the sprawling options I'm going to let this project rest right now and will return to it with Missy when we're ready to see it through! 

In the future, I'd like to return to when I have more ReactJS experience under my belt so that we can restructure this to be optimized. I'm going to take a pause from working on this and am going to work on a new, similar web app that for the moment I'm calling: **WHAT SHOULD I PLANT**. This web app aims to be a minimal, clean design that allows a user to get help deciding what to plant at any given moment in the year in North America. I'll be connecting with the [Trefle API](https://trefle.io/) to do this.

The goal for this will be to create a streamlined web app that offers a useful digest of information to gardeners to make informed planting decisions. I picture this could be useful to more casual gardeners that don't necessarily plan out a whole growing season from the beginning.

Weather-Or-Not taught me that things can get hairy, fast! I will be building this new web app primarily as a text based interfact, no CSS. Once I get that output in a form that works I can build out a simple HTML + CSS design with components. 

🌱Let's see how that goes.🌱