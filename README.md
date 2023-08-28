# Momento Pizza Shop Workshop 

This is the workshop from Momento showcasing the capabilities from the cache, topics, and auth services they provide. They will also educate participants on core caching concepts like read-aside caching, write-aside caching, and cache invalidation.

## Test Environment

If you'd like to see the work in progress, [check out the live version](https://main.d2yefyrzx166mp.amplifyapp.com/).

## Versions
 * 1.0
    * Initial Release:
    Overhauled to add prescriptive guidance. Improved readability and ease of use by making the base more templatized.


## Description

 This is the base repo for building workshops with AWS. It utilizes the Hugo framework which involves simple mark down and HTML elements.

 In this workshop you are going to learn how to plan, build, and launch an AWS workshop.

 ## What is a workshop?

 An AWS workshop is a tool used to educate users and end customers on how to leverage partner solutions on their AWS workload. What better way to learn than to let customers get hands on with building or instrumenting products or services in an actual AWS environment? A workshop format is used because it scales well, meaning you can deliver the content and message whenever and wherever the customer happens to be: whether the customer is at work, home, or at an AWS event, they can get hands on and learn about building products and solutions.

 ## High level Planning

 While creating your workshop you will want to think about what you want to accomplish and how you want to educate users about your product. Depending on your needs you will either build a new workshop or use an existing one and modify it as you see fit.

 You will want to create a high-level plan for your workshop and determine the problem you are looking to solve. Identifying key concepts that you want the customers to learn about is also ideal. Then outlining what components and AWS services that you are going to utilize is also a key step. This will play a big role in creating the workflow that your customers will be following along with during the workshop. The workflow should be presented as a story and have an introduction, an educational body, and a conclusion that ties all the pieces together. A cleanup section will follow suit so that the customers can make sure their environments will not be charged after they finish the workshop. Determining what kind of event this workshop will be presented at is also rather important as it will help you have a way to capture leads which should be the end goal in mind. (More details are covered in the workshop itself)

 ## Types of Events
 
 Identifying whether your workshop will be a self-paced workshop or an AWS hosted event will be crucial in your planning as well. If it’s a self-paced workshop then having highly contextualized sections will be very important as there won’t be anyone there to answer questions. Making sure that the workshop itself has all relevant information or that clear references have been outlined for any documentation that will be needed to successfully complete the workshop are crucial to the workshops overall success.

 ## Build

 In this section, you will be setting up your workflow, edit and build, test your environments, and publish the workshop if everything is prescriptive enough. Follow along to learn how to complete all of these tasks. 

 To run locally, make sure you have [Hugo installed](https://gohugo.io/installation/) on your machine. After installation, you can build the static site in a terminal located in the root directory with the following command:

 ```bash
 hugo
 ```

 To run the site, use this command:

 ```bash
 hugo server
 ```

 This will start the site locally, usually on `http://localhost:1313` when the port is available.

## Launch

 With your workshop now being published, you can now identify some key got to market activities. 

## FAQ

 Commonly asked questions along with tools, tips, and samples that might be relevant to your workshop. Modify this FAQ section as you best see fit for your specific workshop and your customer base. 

## Authors

Contributor names and contact info:

* Pete Naylor (@pj_naylor)
* Allen Helton (@allenheltondev)  

## License

This project is licensed. See the LICENSE.md file for details

## Acknowledgments

* Markdown cheat sheet (https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
* Learn theme markdown (https://learn.netlify.app/en/cont/markdown/)
* Menu extras and shortcuts (https://learn.netlify.app/en/cont/menushortcuts/) 
* Using Font Awesome Emoji's to help your page pop (https://learn.netlify.app/en/cont/icons/)