---
layout: post
title: "Plotly in Angular: Mocking the Back-end"
categories: [angular, plotly, plotting, data visualization, back-end]
assets: "/assets/plotly-angular-mocking-the-backend"
post_series: "angular-plotly"
repo_url: "https://github.com/mrgne1/mocking-the-backend"
repo_name: "Mocking the Back-end"
---

I know, I know, I know! This is supposed to be about angular-plotly and I promised that were were going to get some more interesting plots. And yet we're still going to have to suffer staring at that demo plot for one post more. 

![data flow diagram](\assets\plotly-angular-mocking-the-backend\data_flow_diagram.png)

I mean really. Look at the figure above. Look at how many components are "mocked" aka not there at all. The stuff in that dashed box isn't there: The data server, the API and the HTTP service. That annoys me and it should annoy you. The HTTP Service and the API both are part of the front-end. To not implement them (at least paritially) is bad form for a front-end development. To implement those, we need to properly mock the back-end server.

That leaves us with the question of what do we do about these missing components. Well, Angular comes prepackaged with an HTTP Service (HttpClient), so we don't really need to implement that. It exists already, we just aren't using it. That leaves us to implement the API and the data server with it's data.

We could create the whole back-end ourselves. Using something like [Django](https://www.djangoproject.com/) or [ExpressJS](https://expressjs.com/) or [Ruby on Rails](https://rubyonrails.org/). Creating our own API and mocking up some data with a few generation functions. However, I am naught but a lowly blogger on the interwebs and you don't pay me enough. Maybe for another time.

We could use some publically available data. Ones with existing API's. A quick ["google search"](https://duckduckgo.com/?q=free+online+data+sources&t=brave&ia=web) reveals several promising sources. One of them is for the World Health Organization (WHO). The WHO has two API's for it's data, one is the [Athena API](https://www.who.int/data/gho/info/athena-api) which is supposed to be a simple API and it looks like it will return data in .xls, .csv, etc. Cool if you want to do some analysis without the coding. The other API is the [GHO API](https://www.who.int/data/gho/info/gho-odata-api). It is more like what were are talking about. You can use URI's to query their database and return data to you. Totally awesome. Google has some data API's and includes a [search](https://www.google.com/publicdata/directory). Amazon, likewise, has the Registry of Open Data on AWS [(RODA)](https://registry.opendata.aws/) (not to be confused with [Rota](https://www.aytorota.es/)). All of these are good, however they aren't what I'm looking for.

I want to use custom data on future blog posts. This data doesn't have a free and public API, I've searched. The data is out there, but nobody is hosting it. Since I want my own data, but I don't want to build an entire back-end I've settled on using the npm package [json-server](https://www.npmjs.com/package/json-server). It's a basic data server which implements a RESTful API for providing JSON objects. It's a complete dataserver/API solution, just add data. It's also simple to setup, just a (six-minute video)[https://egghead.io/lessons/javascript-creating-demo-apis-with-json-server] from (egghead.io)[https://egghead.io] and you'll know everything I do. Yay!

Install the json-server module with the command: `npm install -g json-server`. The '-g' will install it globally so it can be run from any folder on your filesystem. Then add the following JSON object to a file called `demo_data.json`. Place `demo_data.json` into the base folder of your angular project. 

{% highlight JSON %}
{
    "lineDemo": [
        {
            "id": 0,
            "x": 1,
            "y": 2
        },
        {
            "id": 1,
            "x": 2,
            "y": 6
        },
        {
            "id": 2,
            "x": 3,
            "y": 3
        },
        {
            "id": 3,
            "x": 4,
            "y": 3
        }
    ],
    "barDemo": [
        {
            "id": 0,
            "x": 1,
            "y": 2
        },
        {
            "id": 1,
            "x": 2,
            "y": 5
        },
        {
            "id": 2,
            "x": 3,
            "y": 3
        },
        {
            "id": 3,
            "x": 4,
            "y": 3
        }
    ]
}
{% endhighlight %}

Now you can start the json-server with the data by running the command: `json-server demo_data.json`. Navigating your browser to `localhost:3000` should show something similar to the following:

![json-server success](\assets\plotly-angular-mocking-the-backend\json-server-running.PNG)

Two components down one to go. We've got the data server running with our data and an API running as well (thank you json-server). Now we need to get the HTTP service up and running as well. As I said before it already exists it's called [HttpClient](https://angular.io/api/common/http/HttpClient). HttpClient allows us to transfer data via the usual HTTP GET, POST, PUT and DELETE messages.

To use the HttpClient we first need to import the `HttpClientModule` from `@angular/common/http`, place it into the list of imports in the `app.module.ts` file. Your `app.module.ts` file should now look like:

{% highlight TypeScript %}
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

import * as PlotlyJS from 'plotly.js-dist-min';
import { PlotlyModule } from 'angular-plotly.js';
import { MrPlotComponent } from './mr-plot/mr-plot.component';
import { HttpClientModule } from '@angular/common/http'; // Add this

PlotlyModule.plotlyjs = PlotlyJS;

@NgModule({
  declarations: [
    AppComponent,
    MrPlotComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    PlotlyModule,
    HttpClientModule, // Add this
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endhighlight %}


With that done we are going to be modifying the `plot-data.service` file. We need to inject the HttpClient into the service, this is done by adding the line `protected http: HttpClient` as an argument of the `plot-data` service's constructor. The `HttpClient` class can be also be imported from `@angular/common/http`.

We can then get our line data by adding the following code to the `data()` function. This code will pull the data from the json-server and then print it to the console so we can look at it.

{% highlight TypeScript %}
this.http.get<any>('http://localhost:3000/lineDemo').subscribe(
    (lineData) => {
        console.log(lineData)
    }
)
{% endhighlight %}

The console output should look like the following:

![lineData in console](\assets\plotly-angular-mocking-the-backend\lineData_in_console.PNG)


Looking at the code that we can see that we are calling the `get()` function on the `http` instance of the `HttpClient` class. This function sends an HTTP GET message to the URL we indicate. We could also use the `put()`, `post()` and `delete()` functions. After we pass the URL 'http://localhost:3000/lineDemo' to the `get()` function we get back an `Observable` from the [RxJS](https://rxjs.dev/api) library. This is a library we are going to have to get familiar with. Angular is deeply entwined with RxJS. Read the [docs](https://rxjs.dev/api).

For our purposes the `Observable` object is an asynchronous source of our API data. The `HttpClient` contacts the json-server on the other end of the url and waits for it to respond. When the json-server responds with the API data, that data is passed through the `Observable` to our code in the `subscribe()` function. And right now our code will print it to the console.

A few of you keen-observers will have noticed by now that the json-server is returning our data as a list of objects where each object contains the x, y pair. You will also have noticed that PlotlyJS wants a list of x values and a list of y values. We should transform one to the other. RxJS allows us to do this nicely with the `Observable.pipe()` function and the RxJS operators specifically the `map` operator. This is analogous to the JavaScript/TypeScript `Array.map()` function. I could try and explain it but it's best to just look at the code.

{% highlight TypeScript %}
this.http.get<any>('http://localhost:3000/lineDemo').pipe(
      map((lineData: {id: number, x: number, y: number}[]) => {
        return {
          type: 'scatter',
          mode: 'lines',
          x: lineData.map((item) => item.x),
          y: lineData.map((item) => item.y),
          line: {
            color: 'red',
          },
        }
      })
    ).subscribe(
      (lineTrace) => {
        console.log(lineTrace)
      }
    )
{% endhighlight %}

And the console output should look like:

![lineTrace in console](\assets\plotly-angular-mocking-the-backend\lineTrace_in_console.PNG)

Nice. Now we just need to return the data to our `mr-plot` component. That creates a problem. Our `data()` function in the `plot-data.service` returns the data straight away and the `HttpClient` needs to wait until the server responds. So we need for the `mr-plot` component to accept the `HttpClient`'s returned `Observable`. 

First let's modify the `plot-data` service to return the `HttpClient` `Observable`. This will create errors. Errors we will fix by modifying the `mr-plot` component. The new `plot_data.data()` function will now be:

{% highlight TypeScript %}
data(url: string, parameters: {[key: string]: string}): Observable<any> {
    return this.http.get<any>('http://localhost:3000/lineDemo').pipe(
      map((lineData: {id: number, x: number, y: number}[]) => {
        return [
            {
                type: 'scatter',
                mode: 'lines',
                x: lineData.map((item) => item.x),
                y: lineData.map((item) => item.y),
                line: {
                    color: 'red',
                },
            }
        ]
      })
    );
  }
{% endhighlight %}

To fix our `mr-plot` component we will change the whole class to the following: 

{% highlight TypeScript %}
@Component({
  selector: 'app-mr-plot',
  templateUrl: './mr-plot.component.html',
  styleUrls: ['./mr-plot.component.scss']
})
export class MrPlotComponent implements OnInit {
  
  @Input() url: string = '';
  @Input() parameters: {[key: string]: string} = {};

  graph: {data: Object[], layout: Object} = {
    data: [],
    layout: {
      width: 320,
      height: 320,
      title: "A Fancy Plot",
    },
  };

  constructor(
    protected plot_data: PlotDataService,
  ) { }

  ngOnInit(): void {
    this.loadData();
  }

  loadData(): void {
    this.plot_data.data(this.url, this.parameters).subscribe(
      (lineTrace) => {
        this.graph.data = lineTrace;
      }
    )
  }

}
{% endhighlight %}

Our angular-plotly plot should now look like:

![lineTrace only plot](\assets\plotly-angular-mocking-the-backend\lineTrace_only_plot.PNG)

Notice that we've moved the PlotlyJS `layout` object back down to the `mr-plot` component. We had originally moved it up to the `plot-data` service because I thought it would be better there. You live you learn and we learned to plan better. On the plus side, we can now eliminate the `lineTrace`, the `barTrace` and `layout` objects from the `plot-data` service. 

We now have the data server running, the API up and running and the HTTP service running. And yet we're not done. We also need to get our `mr-plot` object to show the line trace as well as the bar trace. Since the plot data is at two different URL's it looks like we will have to subscribe to two different `HttpClient` `Observables` and manage their insertion to the `graph.data` array in the `mr-plot` component. RxJS to the rescue. RxJS has the `zip()` function, this function will combine the data from multiple `Observables` into a single array. This means we can create one `HttpClient` `Observable` for the `lineData` and another for the `barData` and then zip them together into a single array. In the `plot-data` service this will look like:

{% highlight TypeScript %}
data(url: string, parameters: {[key: string]: string}): Observable<any[]> {

    let line = this.http.get<any>('http://localhost:3000/lineDemo').pipe(
      map((lineData: any[]) => {
        return {
          type: 'scatter',
          mode: 'lines',
          x: lineData.map((item) => item.x),
          y: lineData.map((item) => item.y),
          line: {
            color: 'red',
          },
        }
      })
    );

    let bar = this.http.get<any>('http://localhost:3000/barDemo').pipe(
      map((lineData: any[]) => {
        return {
          type: 'bar',
          x: lineData.map((item) => item.x),
          y: lineData.map((item) => item.y),
        }
      })
    );

    return zip(line, bar);
  }
{% endhighlight%}

The angular-plotly plot should now look like:

![complete demo plot](\assets\plotly-angular-mocking-the-backend\complete_demo_plot.PNG)

Okay, the plot is looking 'nice' again. However, the `plot-data` service is still kind of poopy. We still aren't using the `url` or the `parameters` in the `data()` function call. We can abstract the code in the `data()` function. Also, we can see that to support our demo plot we need to pass two different URL's. So the `data()` function should take a list of URL's. Finally we can see that both URL's get a different mapping function. So we should support different mapping functions for each URL.

Let's do the mapping functions first. We'll create an object in the `plot-data` class where the keys are the URL's and the values are those `map()` calls. It looks like:

{% highlight TypeScript%}
  private maps: {[url: string]: OperatorFunction<any[], any>} = {
    'http://localhost:3000/lineDemo': map((lineData: any[]) => {
      return {
        type: 'scatter',
        mode: 'lines',
        x: lineData.map((item) => item.x),
        y: lineData.map((item) => item.y),
        line: {
          color: 'red',
        },
      }
    }),
    'http://localhost:3000/barDemo': map((lineData: any[]) => {
      return {
        type: 'bar',
        x: lineData.map((item) => item.x),
        y: lineData.map((item) => item.y),
      }
    }),
  }
{% endhighlight %}

Make sure you get the type for maps up there, otherwise you'll see some errors. Now we can change the `get()` calls to the following:

{% highlight TypeScript%}
    let line = this.http.get<any>('http://localhost:3000/lineDemo').pipe(
      this.maps['http://localhost:3000/lineDemo']
    );

    let bar = this.http.get<any>('http://localhost:3000/barDemo').pipe(
      this.maps['http://localhost:3000/barDemo']
    );
{% endhighlight %}

Much cleaner and now that it is cleaner, I think you can see where this is going. These two `get()` calls are identical except for the URL. So we can change the whole `data()` function call to:

{% highlight TypeScript %}

  data(urls: string[], parameters: {[key: string]: string}): Observable<any[]> {
    return zip(urls.map((url: string) => this.http.get<any>(url).pipe(this.maps[url])));
  }
}
{% endhighlight %}

Make sure you update all the urls everywhere. That means `/src/app/plot-data.service.ts`, `/src/app/mr-plot.component.ts` and `/src/app/app.component.html`. It should be pretty clear what you need to update, but if you are confused or something isn't working right, check this post's github repo. That should help.

We have one thing left to do for this blog post and that is to utlize the `parameters` argument of the `data()` function in the `plot-data` service. The `HttpClient.get()` function takes the parameters as an `HttpParams` object. An example is best. Let's try to pull only the last 3 data items for our plot data. Now, each data item has an id and according to the json-server [specification](https://www.npmjs.com/package/json-server#operators) we can select for the items with id's 2, 3 and 4 by passing parameters as 'id_gte=2&id_lte=3' (read the docs). To do this with `HttpParams` we create a new variable `params` and then we append each of the parameters as key value pairs. We will then pass this into the `HttpClient.get()` function. The code looks like: 

{% highlight TypeScript %}
    let params = new HttpParams()
    params = params.append('id_gte', 1);
    params = params.append('id_lte', 3);
    return zip(urls.map((url: string) => this.http.get<any>(url, {params}).pipe(this.maps[url])));
{% endhighlight %}

Now to finish this part of the project, we can pass the parameters in to the `mr-plot` component inside the `src/app/app.component.html` file and utilize them in the `plot-data` service's `data()` function. You should be able to figure this out, I believe in you. However, if you don't know what to do, the github repo is there for you.

By now we've got a pretty good environment built for our plotting. We have a back-end data server with custom data, a feature-rich API, an HTTP service and a front-end plot-data service. The whole data pipleline up and running. It only took 3 posts to get here. In the next post, let's do something more interesting or at least something with PlotlyJS.