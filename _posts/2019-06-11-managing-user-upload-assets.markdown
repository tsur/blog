---
layout: post
title:  "Managing User Uploaded Assets"
category: Tech
author: Zuri Pabón
---

This document briefly explores different solutions for managing the process of uploading and downloading media resources for users

# Uploading process

The traditional uploading process for assets is managed by the backend. The client uploads the resource to the backend using an HTTP multipart request. The backend, in turns, uploads the resource to the storage service (i.e. AWS S3). 

This approach requires an extra round trip which increases the data we are transferring by 2x times, negatively impacting on the performance and degrading the user experience. 

We also had to face random, hard to track and debug issues regarding stream pipes forcing us to wait for the stream to finish before being able to start emitting upload stream data again to the micro-service.

In order to overcome those issues, we proposed uploading the resources straight away from the client to the storage service. How does it work? When the user uploads a resource, the client requests a signed file upload URL to the backend. 

It is managed at the backend for security reasons. Generating a signed URL does not require any I/O operation and no additional request to AWS is actually issued so it resolves pretty fast. Once the client gets the signed URL from the backend, it performs a multipart upload to the signed AWS S3 URL. After implementing this approach, we made some tests to measure the performance gain. Details below.

Benchmark scenario: Uploading 5 pictures (JPG - 80K average size), Iterations: 10

Metrics used are the maximum (worst) from 10 iterations, so this reflects the worst case scenario for the given iterations.

* Case A. uploading and creating picture entity in storage service: **23s**
* Case B. uploading to S3 and creating picture entity in storage service: **3s**
* Case C. uploading to S3 with no storage service interaction: **1.22s**

* diff (A-B) = 19349.0009765625ms/19.3490009765625 - **B is aprox. x6 times faster than A, about 16.02% A time**
* diff (A-C) = 21821.27587890625ms/21.82127587890625s - **C is aprox. x18 times faster than A, about 5.3% A time**

Case A is our current implementation and B is the solution we migrated to. The C scenario could be improved further by enabling the S3 accelerated transfer service, but the bottleneck is on the backend HTTP requests.

Creating a picture entity in the storage service from backend takes about x2 times the uploading time. The uploading time is about 1.2s for this benchmark scenario whereas creating the picture entity in the storage service from backend is about 2.4s. This includes making the graphql request to backend, two requests to storage rest API (one for creating picture entity and another for assigning the picture to the user) and saving to the local database.

Note: Please consider this benchmark scenario has very few iterations, meaning statistical significance can be low.

# Downloading process

Once the original resource is uploaded from the client, we can download the original picture from the S3 URL we used to upload the picture. Using CloudFront along with S3 would help optimizing resources delivery performance since CloudFront serves content through a worldwide network of data centers improving the performance by caching and serving content closer to where users are located.

Regardless of using CloudFront or not, most of our resources are pictures, so it becomes required a process to generate thumbnail images in order to serve reduced file size pictures and get faster loading times. 

Instead of processing and resizing images into all necessary sizes as part of the uploading process or as part of a background job, we made the choice to generate thumbnails dynamically on the fly due to the several advantages it provides like higher flexibility (if a redesign requires it, we can add new dimensions and increase/decrease resolution on the fly), reduced storage costs or resilience to failures (if the thumbnail generation fails, we can recover by issuing a new request to the resource). Below we explore different solutions to generate thumbnail pictures on demand. 

## Custom Image Processing Server 

The first solution is to use a HTTP image processing server as a micro-service that exposes an endpoint to generate thumbnails images on demand. It receives the original image URL and different options to specify the size and other possible image transformations.

As part of this solution, we implemented a basic HTTP image processing server in go (you can check the source code here https://github.com/zuripabon/img-proxy-thumbnails) which exposes a simple API Rest with a GET /thumbnail endpoint. The client code may request this endpoint by passing in the thumbnail size and the URL from which the thumbnail image will be generated as query params. It generates the thumbnail dynamically in memory and returns it back as the HTTP response.

This solution gives us more control and enables us to customize and adapt it better to our own needs. For instance, we could support formats which are not commonly supported like HEIC images. However, this is not actually our market target and making it a robust solution would demand to spend more resources in order to efficiently maintain the project. 

Implementing advanced features such as caching, error recovery or security (picture bombs) are time-consuming and require dedicated team effort to accomplish it. 
Besides that, the server is implemented in go which is more suitable for this sort of computation intense tasks but will require a context switch for many developers who are used to work with other languages as node.js meaning there is a learning curve process for the team to adapt to in order to be able to contribute to the project.

## Amazon Serverless Image Handler

An alternative to using our own image processing server solution would be to use the Serverless Image Handler (https://aws.amazon.com/solutions/serverless-image-handler/) solution. The working principle is similar to the image processing server in the sense it generates a thumbnail image on demand, but it is an end to end solution ready for production.

From our client code, all we have to do is to generate a CloudFront URL based on the S3 bucket and the picture key we used when uploading the resource and then rendering it from an img HTML tag. 

When the browser requests the img src resource it will not be found on CloudFront, so it redirects the request to a lambda which creates the thumbnail picture and uploads it to S3, then it redirects back again to CloudFront which now finds the thumbnail picture and return it back to the browser. Next time the thumbnail picture is requested is served directly from CloudFront.

The main advantage it provides from a development point of view is that it would increase dev team productivity as we can focus on other tasks instead of spending time in maintaining and improving the thumbnail generation process. 

From the DevOps point of view, this solution integrates pretty well with all AWS ecosystem and setting this up is a 30 minutes process according to AWS documentation. 

This is a solution we can use across all the dev teams in the company as it automatically scales resources on demand. One important drawback, however, would be in the case of needing custom modifications. 

The only critical one so far is supporting HEIC format. Despite Serverless Image Handler accepts a rich variety of formats as png, jpg, tiff, web, among others, heic/heif is not included. Note we could also need to implement some removal policy for thumbnails pictures which are no longer requested

## ImgProxy Server

Finally, an as alternative to Amazon Serverless Image Handler, we could use the open source imgproxy server (https://evilmartians.com/chronicles/introducing-imgproxy), which is a well tested, secure and fast solution but would have the same issues regarding customization and a serverless solution as the one provided by AWS is probably far more reliable and cheap than dedicated servers. 

The good news concerning the customization issues is that both Amazon Serverless Image Handler and the open source Imgproxy server are based on libvips and as long as version 8.8.0 is released (https://github.com/libvips/libvips/milestone/6), both of them will support heic/heif format (https://github.com/lovell/sharp/issues/1105, https://github.com/imgproxy/imgproxy/issues/140) which is not currently supported natively by browsers (https://caniuse.com/#feat=heif)  

As a final conclusion, I would discourage using our own custom thumbnail image processing server implementation and would recommend instead to use Amazon Serverless Image Handler solution for a long term better solution, which is easier to maintain and to scale up, which also provides a more reliable architecture and integrates out of the box with S3 and CloudFront, falling back to the open source imgproxy server (https://github.com/imgproxy/imgproxy) or to our own image processing server implementation in case we face some unexpected technical or non-technical limitations with Amazon Serverless Image Handler solution. 

Additional Notes:

most of our pictures will be displayed at a size of 288x264 pixels in the landing cards as specified in design.

Amazon Serverless Image Handler estimated costs for 1 million images processed, 15 GB storage and 50 GB data transfer is about $13 (https://docs.aws.amazon.com/solutions/latest/serverless-image-handler/overview.html)
