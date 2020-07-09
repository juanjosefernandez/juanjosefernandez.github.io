---
title:  "Weather-Or-Not"
author: Juan Jose Fernandez
comments: true
layout: post
date:   2020-07-07 00:04:51 -0400
categories: programming
tags:
- weather-or-not
- missy moreno
- React
- OpenWeather API
- MapBox API
- api 
image: /assets/images/weatherornot.gif
vertical: Code
excerpt: Death is around the corner. How we come to terms with, that is the business of our living.
# excerpt_separator: "<!--more-->"
summary: Death is around the corner. How we come to terms with, that is the business of our living.
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

Earlier this summer my buddy [Missy Moreno](https://www.missymoreno.com/) and I decided to make inroads in learning how to work with React and APIs. We decided to make a cheerful weather web app that had some kind of suggestion system based on the weather forecast at a given location.

Having just recently learned how to write successful code in ReactJS, we dove in and got to work. We had plenty of messy false before we landed on a good component structure that allowed for us to do what we wanted. Not having ever used an API on my own, I thought it would be good to start with some guard rails. We used [this tutorial from RapidAPI](https://rapidapi.com/blog/weather-app-react/). It didn't connect to the endpoints that we needed it to, but that was ok, we needed to figure out how this all worked!

After we got that working we had their web-app. We took a step back and discussed what we needed to do differently. Notably, we wanted to get the weather conditions for the next 5 days, which required using a different API which required latitude and longitude. So, we doctored up our code to hit the appropriate API endpoints and we were in business.

Though we got that working, we were manually entering Pittsburgh's latitude and longitude to test. No good! Naturally that's not a reasonable way to have the user enter their location. Turns out we needed to connect to another API to help us transform city names into latitude and longitude coordinates! 

[Mapbox](https://docs.mapbox.com/api/) was the ticket. We used the [forward geocoding](https://docs.mapbox.com/api/search/#forward-geocoding) feature of the API. At this point we had the intended usage for the applications user interface. I coded up the content of the Strip components so that they were filled with High and Low temperatures, along with a weather condition description.

Because Javascript is a collection of asynchronous processes, it was necessary to chain the calls in such a way where you never call the Openweather API until you've gotten a successful response from the Mapbox API. As you'll see below the two fetches are linked by a callback that you can see in line 29???? I had a lot of fun figuring out how to make this work. LINK TO A GOOD BLOG POST - https://gomakethings.com/how-to-use-the-fetch-method-to-make-multiple-api-calls-with-vanilla-javascript/

REMOVE THE API KEYS

Here's a look at the way that this works!

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
    fetch(`https://api.mapbox.com/geocoding/v5/mapbox.places/${city}.json?access_token=pk.eyJ1IjoianVhbmpvc2VmZXJuYW5kZXoiLCJhIjoiY2tiaTRyNWM4MGJ1NTJ5bWx2Yzd5a3E3eSJ9.3nNSmXu7AqLrHF-MAepd-A`).then(function (response) {
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
        return fetch(`https://api.openweathermap.org/data/2.5/onecall?lat=${mapBoxData.features[0].center[1]}&lon=${mapBoxData.features[0].center[0]}&exclude=minutely,hourly&units=imperial&appid=de21f1eaf5bf29f1eb059f7f97f70b23`);

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

blah blah blah

We passed the OpenWeather response object over to the conditions component, which contains the strip components.

    <Conditions
    // this component holds our strips.
    responseObj={responseObj}
    error={error} //new
    loading={loading} //new
    safe={safe}
    />

fart blah blah blah

    <Strip whichStrip="StripOne" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={0}/> 
    <Strip whichStrip="StripTwo" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={1}/> 
    <Strip whichStrip="StripThree" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={2}/> 
    <Strip whichStrip="StripFour" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={3}/> 
    <Strip whichStrip="StripFive" responseObj={props.responseObj} safe={props.safe} loading={props.loading} day={4}/> 

and each strip is composed as below 

    <div > 
        <container>
            <section className={props.whichStrip}>
            {/* <p>Hi Temperature, Lo Temp, Description, Day of week and date:</p> */}
            {/* where we use the data from the API call, only render it IF the API call has been successful */}
            {/* <p>Safe/Unsafe: {props.safe ? 'safe' : 'unsafe'} </p> */}
            <p>{props.safe ?  "Lo: " + JSON.stringify(props.responseObj.daily[props.day].temp.min) + "°F" : ""}</p>
            <p>{props.safe ? "Hi: " + JSON.stringify(props.responseObj.daily[props.day].temp.max) + "°F"  : ""}</p>
            <p>{props.safe ?  "Main Description:" + JSON.stringify(props.responseObj.daily[props.day].weather[0].main) : ""}</p>
            <p>{props.safe ?  "Sub Description: " + JSON.stringify(props.responseObj.daily[props.day].weather[0].description) : ""}</p>
            <div>{props.safe ? <img src={"http://openweathermap.org/img/wn/"+ iconId + "@2x.png"}></img> : ""}</div>
            {/* {props.safe ?  JSON.stringify(props.responseObj.daily[props.day].weather[0].icon) : ""} */}     
            </section>
        </container>
    </div>

fart

It's not the best code, but I want to show my work here. It's clunky and could work far more efficiently. But it works! And that's what counts for me and Missy in our journey of knowledge. Missy has been having lots of fun playing with the styling. It's wacky and I love it.

Missy has a big vision of what she wants this web app to be in the future. My focus was on what I shared above, learning how to make API calls correctly and using components in ReactJS.

In the future, I'd like to return to when I have more ReactJS experience under my belt so that we can restructure this to be optimized.

>We need to build a table of activity suggestions that are offered to the user for suggestions as to what they might think to do in the upcoming days. The idea would be to have a collection of rainy weather activities that are randomly selected. 

>The idea is to create a table of conditions for example if it is  sunny but the high temperature is less than 35 degrees, it makes sense to suggest going outside as it's likely going to be a cool and crisp day. Say it is above 80° and is "heavy rain", it'll be a muggy day where you'll likely be cooped up inside. Board games with cool drinks are likely a good suggestion. This is something that Missy and I need to sit down to do. Another thing we were thinking of was connecting a recipe API that would be called and passed different variables based on the weather conditions. As you can see the weather can trigger a lot of different responses. Rather than lose myself in the sprawling options I'm going to let this project rest right now and will return to it with Missy when we're ready to see it through!

- - - - - - - 

I'm going to take a pause from working on this and am going to work on a new, similar web app, **WHAT SHOULD I PLANT**. This web app will aim to be a minimal, clean design that allows a user to get help deciding what to plant at any given moment in the year in North America. I'll be connecting with the [Trefle API](https://trefle.io/) to do this.

The goal for this will be to create a streamlined web app that offers a useful digest of information to gardeners to make informed planting decisions. I picture this could be useful to more casual gardeners that don't necessarily plan out a whole growing season from the beginning.

Weather-Or-Not taught me that things can get hairy, fast! I will be building this web app primarily as a text based interfact, no crazy CSS. Once I get that output in a form that works I can build out a simple HTML + CSS design with components. 