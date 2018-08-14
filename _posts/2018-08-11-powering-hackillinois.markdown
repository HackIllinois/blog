---
layout: post
title: "Powering HackIllinois"
date: 2018-08-11 09:00:00
categories: technical
tags: HackIllinois2018
author: Arnav Sankaran
---

Today we'll be taking a look behind the scences at the software which powers HackIllinois. If you've attended the event in the past you've probably seen our website and mobile apps. What you might not have seen though is the [HackIllinois API](https://github.com/HackIllinois/api) which these applications interact with to deliver you your content. The API has been through a few iterations over the years. We'll be exploring the motivations for these iterations and current version that we will be deploying for HackIllinois 2019.

### Purpose
Before going into the details of how the API is implemented let's first consider: what is the purpose of the API? The HackIllinois API exists to provide a simple, easy-to-use REST interface for managing user and event data. The website and mobile apps should be able to interact solely with the API and have any external interactions will be mediated via the API.

### Defining the Requirements
Just like any other project, the first step is to define the goal of the finished product and then to define the requirements the product must meet to acheive it's goal. The goal of the HackIllinois API is to manage all user and event data. Based off this goal we can extract out a few core requirments for the API:
	- Allow hackers to login and create an account
	- Allow hackers to register for HackIllinois
	- Allow staff to accept, waitlist, and reject applicants
	- Allow hackers to rsvp to HackIllinois once accepted
	- Allow staff to checkin hackers on the day of the event
	- Allow staff to manage the schedule for events during HackIllinois
	- Allow staff to track which hackers have taken part in events
There are a certainly more features which would be very useful for both staff  and hackers at HackIllinois, but these are the core requirements we will start off with. Once completed, we can explore more advanced functionality.

### Implementation Strategy
As mentioned at the begining of the post, there have been a few iterations of the API over the years. The purpose and requirements of the API over the years has remainds relativley constant, however the implementation has greatly different from iteration to iteration. Each iteration of the API aims to innovate and take a new approach to solving problems which arose in the past. [Last year's API](https://github.com/HackIllinois/api-2018) was implemented as a monolithic application using nodejs and MySQL for the database. It worked and cewrtainl met the requirements defined for HackIllinois 2018, however some issue we ran into during the development was the lack of compile time type safety and annoyance of building a new database migration for any minor change to the defined scheme. This year we aimed to reimplement the API from scratch and resolve these issues.

Based on this we decided to implement the API using a microservices based architecture in Go using MongoDB for the database. The primary strength of this architecture is that each core requirment can essentially be implemented as it's own service allowing us to define the public interface for each service and leave the implementation details up to each developer. We could even have each service written in a different language with a different database if we wanted, but for the sake of some consistency in the API we have chosen to use Go and MongoDB for every service. The reason for choosing MongoDB over MySQL was the ability to rapidley change the scheme in MongoDB due to it's document based storage format. This eliminates the need to create database migrations everytime we make a small scheme change. And scheme changes do happen more frequently than you might think due to the team making changes the registration, admission, and rsvp process.

### The Microservices Architecture
To provide a single clean interface to the API we have an API gateway that redirects incoming requests to the correct service and then passes the response from the service back to the user.

#### The Gateway
The gateway performs authentication verification, identity managment, and manages permission controls. Users send their token to the gateway with each request. The gateway decodes the token to verify it is valid and not expired. Then the gateway checks if the user's roles match the roles allowed to access the specified endpoint. If the user does not have the roles, the gatway returns a 403. This allows us to restrict certain features of the API to Admins and Staff members. Assuming the token is valid and active and the user the proper permissions the gateway modify the request by injecting a `HackIllinois-Identity` header into the request containing the unique id of the user which was embedded within the token. This unique identity allows services to know who made the request without needed to decode the user's token and check the user's permissisons. This modified request is then forwarded to the correct service. The result from the service is passed back to the user. The proxying portion of the gate way is handler by [Arbor](https://github.com/arbor-dev/arbor), a framework built and maintained by [ACM@UIUC](https://acm.illinois.edu) Projects.

#### The Services
Currently the HackIllinois API has the following services: auth, user, registration, upload, decision, rsvp, checkin, mail, event, and stat. These services communicate over http to each other. The location of each service is loaded from configuration environment variables. It works best when paired with DNS to allow dynamic service locations. It's fairy easy to what each service does but here is a slightly more in depth description of each service.

##### Auth Service
The auth service allows user to login via a supported OAuth provider, currently GitHub, LinkedIn, and Google and creates an API token for the user containing the information needed by the gateway for authentication. It also manages the roles for each user, allowing users to be granted additional permissions to features of the API.

##### User Service
The user service manages storage of basic user information such as unqiue identifier, name, and email.

##### Registration Service
The registration service manages the registrations for both hackers and mentors. Staff members are able to retreive the registrations of user for review.

##### Upload Service
The upload service allows user to upload files to one of our S3 buckets. It is currently used for uploading resumes during the registration process.

##### Decision Service
The decision service allows staff members to mark an applicant as accepted, waitlisted, or rejected. A senior staff member can then finalize this decision for the applicant.

##### Rsvp Service
The rsvp service allows accepted hackers to rsvp to HackIllinois. This enables the HackIllinois team to know how many people will be attending.

##### Checkin Service
The checkin service is used on the day of the event by staff to checkin hackers to the event. The service is built to integrate with QR codes and QR readers on the frontend allowing the entire checkin process to be completed by a staff member scanning a hacker's QR code.

##### Mail Service
The mail service allows staff members to send an email to a user or a group of users. When a user is given a decision in the decision service, they are also added to a mailing list in the mail service. This is particularly useful during the admissisons process. The mail service also generates substitutions for templated email. For example the email template may say `{ name }` in it and mail service will recognize that and insert the user's actual name into the email.

##### Event Service
The event service manages the data for events which can be created by staff members. It also allows staff members to scan a user's QR code and mark them as having attended the workshop, talk, or meal.

##### Stat Service
The stat service aggregates statistics from all of the other service. Any service can register it's statistics endpoint with the stat service and it will be available for Admins to query through the stat service.

### Preventing Regressions During Development
When working a fairly large system with multiple developers it is important to have an enforable way to prevent regressions to the functionality of the project. Having continuous testing on each commit and pull request is simple way to acheive this goal. For continous testing, we chose to use Travis mainly because it integrates nicely into GitHub and is free for open source projects. Any pull request into staging or master branches of the repository require all Travis checks to pass. It also requires a review from another developer. Pull requests reviews are important for maintaining code quality and for catching an potential errors that the testing suite may be able to catch.

### Deployment Strategy
Given that the API is using a microservice based architecture we have a number of possible deployment options. Our goal for deployment are to setup a continuous deployment pipeline such that the latest commit on the master branch is always running in production. We also want to be able to scale up or down the API to accomodate for the increasing amount of traffic we receive as the event approaches. Based on these goals we have decided to use AWS's Elastic Container Service. This allows to us start up each service in it's own container which can independenently be scaled. It also allows us to take advantage of AWS's Route 53 DNS based service discovery which integrates very well into our cross service communication pattern. Each serivice registeres itself with the DNS provider as {service-name}.api. So the auth service would always be available at auth.api and other service can rely on the DNS provider to give them the current IP of the container running the auth service.

Now that we have a target to deploy to, we need to consider how are we going to continuously deployment the latest commit on master to production. As mentioned previously we use Travis for continuous testing on each commit and pull request. We decided to also use Travis's deployment feature. One issue though... AWS Elastic Cluster Service isn't a directly supported deployment target. If you search online you'll find some scripts for travis that install the AWS command line tools and use them to deploy the application. That certainly works but we felt that there were other solutions that didn't involve custom Travis deploy scripts. The end result to create a two stage deployment pipeline. The first stage of the pipline is using Travis to build the binaries for each service. Travis then creates a seperate zip archive for each service containing the service binary, a Dockerfile for containerization, and an AWS buildspec.yml file which will be important for stage 2 of the pipeline. All of these zip files get uploaded to an AWS S3 bucket to complete stage 1 of the pipeline. Stage 2 of the pipeline is an AWS CodeDeploy Pipeline which is set to watch the locations we uploaded the zip files to. The CodeDeploy Pipeline takes each zip file, unpacks it to a build server and builds a container for the service based on the Dockerfile and builspec.yml files in the archive. The resulting container is then uploaded to AWS's Elastic Container Registry and the Elastic Container Service is told to redeploy the service with the latest updated container. AWS's Elastic Container Service ha the ability to do green-blue deployment which we take advantage of to have zero downtime deployment. When the new container is deployed it is brought online first behind a load balancer which drains connections to the old container and switches them over to the new container. Once the new container is fully operational the old container is torn down bring us back to a steady state until the next deployment.

### Building a Generic API
While the goal of the API is to power HackIllinois, the ability to login, register, get accepted, and checkin to an event are common to most hackathons and many other events. As a result the API was designed to be as generic as possible allowing other events to potentially use the HackIllinois API for their event with minimal changes. This year [Reflections | Projections](https://reflectionsprojections.org/), the annual technology conference organized and run by students at the University of Illinois at Urbana-Champaign, will be using the HackIllinois API to manage registration, checkin, and event tracking. In the future we hope to see other hackathons using the HackIllinois API to manage the data for their event.

### Leveraging Technology to Deliver a Better Experience
As we are wrapping up this post, I think it is important to take a moment and reflect on why we are building this API. The end goal of everything we do as HackIllinois staff members is to provide a better experience for hackers, mentors, and sponsors and HackIllinois. Last year the checkin process for hacker was slow and frustrating for many people. The checkin service this year was designed to eliminate that issue. We identified that needing to manually lookup each hacker based on their GitHub username or email was a slow process. To eliminate this bottleneck the API was designed to work with a QR based system on the frontend. All hackers will have a QR code that staff members can scan to check with their QR scanner. Both the QR code and the scanner for staff will be available in the HackIllinois mobile applications. Upon scanning a hacker's QR code the app will checkin the hacker with the API. This is a much faster process than old manual lookup process.

Another service that underwent a massive restructuring was the events service. In the past we were only able to scan hackers into a single event at a time and the list of hackers which attended each event was lost at the end of the event. The event tracking systems this year allows us to scan hackers into multiple events at once and stores list of hackers that attended each event. This allows us to determine the events that are popular with hackers and provide more of them in the future. It also allows us to scan hackers in for getting food accuratley track the amount of food being eaten and number of people eaten food at each meal. With this information we can inteligently make decision about how much food we need at each meal to ensure everyone is well fed and that hackers don't have to wait too long for food to arrive.

### Contributing to the API
If you're interested in working on the API after reading this post, we be glad to have your help. The API is open source and available on GitHub, [here](https://github.com/HackIllinois/api). Check out the issues section and see if there is anything there at interests you and leave a comment. If you are a student at the University of Illinois Urbana-Champaign, come to our meetings and join HackIllinois staff. We are looking for not only developers to work on the API, but also on the website, Android, and iOS mobile applications. We will be annoucing more information about our meeting time at the ACM Open House.
