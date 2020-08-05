# Architecture Overview of the ParksMap Application

This lab introduces you to the architecture of the ParksMap application used throughout this workshop, to get a better understanding of the things you’ll be doing from a developer perspective. ParksMap is a polyglot geo-spatial data visualization application built using the microservices architecture and is composed of a set of services which are developed using different programming languages and frameworks.
![Application architecture](https://github.com/bhandaru/nationalparks-labs/blob/master/images/roadshow-app-architecture.png)

The main service is a web application which has a server-side component in charge of aggregating the geo-spatial APIs provided by multiple independent backend services and a client-side component in JavaScript that is responsible for visualizing the geo-spatial data on the map. The client-side component which runs in your browser communicates with the server-side via WebSockets protocol in order to update the map in real-time.

There will be a set of independent backend services deployed that will provide different mapping and geo-spatial information. The set of available backend services that provide geo-spatial information are:  
   * WorldWide National Parks
   * Major League Baseball Stadiums in the US
   * International Airports
   * Earthquakes epicenters in New Zealand in November 2017
   * Etc

The original source code for this application is located [here](https://github.com/openshift-roadshow/).

The server-side component of the ParksMap web application acts as a communication gateway to all the available backends. These backends will be dynamically discovered by using service discovery mechanisms provided by OpenShift which will be discussed in more details in the following labs.

<p align="center">
  <a href="/03%20-%20Exploring%20CLI%20%26%20Web%20Console.MD">Previous Exercise</a> &nbsp;|
  &nbsp;<a href="/README.md">Table of Contents</a> &nbsp;|
  &nbsp;<a href="/05%20-%20Deploying%20a%20container%20image.MD">Next Exercise</a>
</p>
