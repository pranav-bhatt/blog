---
layout: post
title:  "Creating a CoAP endpoint for the Drogue Cloud project"
author: Pranav
categories: [ Technical ]
image: https://52north.org/wp-content/uploads/2020/02/anmerkung-2020-02-21-093222.png
tags: [ google summer of code, gsoc, coap, drogue ]
beforetoc: "Before I begin, I would like to express my deepest gratitude to my mentors Ulf and Jens. I'm sure I troubled them quite a bit, but they were extremely kind and welcoming nonetheless. The amount I have learnt from them cannot ever be summed up by a blog post xD"
toc: true
---
As GSoC 2021 draws to an end, I can safely say that it has been one of the most enthralling experiences of my life. Here is what I've been up to..."

## TL;DR - List of PRs

- [[cloud] RFC0007 - Mappings for Cloud Event Attributes to CoAP Options](https://github.com/drogue-iot/rfcs/pull/8)
- [Revamped CoAP documentation, source code comments and improved error messages](https://github.com/drogue-iot/drogue-cloud/pull/117)
- [coap-endpoint](https://github.com/drogue-iot/drogue-cloud/pull/87)
- [Updated README.md](https://github.com/drogue-iot/drogue-cloud/pull/65)
- [Modified helm charts to facilitate coap-endpoint](https://github.com/drogue-iot/drogue-cloud-helm-charts/pull/1)
- [added coap tests](https://github.com/drogue-iot/drogue-cloud-testing/pull/2)

## About Drogue-Cloud

[Drogue Cloud](https://github.com/drogue-iot/drogue-cloud) is project part of the [Drogue-IoT](https://github.com/drogue-iot) repository. It is an IoT/Edge connectivity layer that allows IoT devices to communicate with a cloud platform over various protocols. It acts as both a data ingestion plane, as well as a control plane. It offers:

- IoT friendly protocol endpoints
- Protocol normalisation based on Cloud Events and Knative eventing
- Managing of device credentials and properties
- APIs and a graphical console to manage devices and data flows

## The project proposal

In Drogue-Cloud, protocol endpoints convert incoming messages from one protocol to another. They are also responsible for forwarding these messages to the appropriate destination (both device-to-cloud and cloud-to-device).

My proposal proposed creating a protocol endpoint for the Constrained Application Protocol (CoAP), which enables simple devices constrained by hardware and low bandwidth, low availability networking.

This endpoint would be capable of satisfying the sample use-case of publishing telemetry data from IoT devices to the cloud. It would also have the ability to send control messages via CoAP to IoT/edge devices.

You can find my proposal [here](https://docs.google.com/document/d/1ycmtKKMFmqqtCOd1mVVy7YGqUFnyoZCY/edit?usp=sharing&ouid=100524191342524711467&rtpof=true&sd=true)

## Road to build an endpoint

### Step 1: Do research on CoAP and see what tooling exists

I first started off by looking at various resources, such as the [CoAP RFC](https://datatracker.ietf.org/doc/html/rfc7252) and existing tools such as [coap-cli](https://github.com/avency/coap-cli); a cli based coap client.

As the codebase of Drogue Cloud is written in Rust, I also took a look at the existing CoAP libraries that would help me achieve my goals. I found the libraries [coap-rs](https://github.com/covertness/coap-rs) and [coap-lite](https://docs.rs/coap-lite/0.4.1/coap_lite/) to be suitable for my use case.

I also realised that CoAP doesn't provide a way to create custom headers due to its constrained nature. Instead, it gives a range of option numbers that one can assign to have custom meanings. 

As the endpoints needed a way to authenticate the devices and a way of sending commands through the response, I created a specification in the [rfcs repository](https://github.com/drogue-iot/rfcs) listing the CoAP option numbers I used, along with their data types and format. You can check out my pull request [over here](https://github.com/drogue-iot/rfcs/pull/8).

### Step 2: Create the endpoint

As the behaviour of CoAP is similar to HTTP, I could take inspiration from the preexisting HTTP endpoint and work my way up from there. I faced some difficulties as there was no framework to handle CoAP requests (such as actix or ntex). However, my mentors always provided valuable suggestions that helped me out when I got stuck.

The endpoint code can be divided into 6 main parts; the request handler, the option parser, the authentication, publish object creation, wait for a command, and response sender. 

The request handler would intercept any requests and call the option parser. The option parser would then extract the necessary information from the request and pass control back to the handler.

The handler would then call a function to pass the information to an authentication service, followed by another call to publish the necessary information to the Kafka Sink. This will also generate a response if the device has asked for it.

Depending on the parameters passed, there is also a command timeout, which, when set, will cause the response to be delay by the set value. If any command is sent to the specific device, it will piggyback on the response. You can check out the respective PRs for this step  [here](https://github.com/drogue-iot/drogue-cloud/pull/87) and [here](https://github.com/drogue-iot/drogue-cloud/pull/65).

### Step 3: Deploy the CoAP endpoint

To deploy my endpoint, I created new Helm charts to expose the correct ports and services. I also had to modify some parts of the build script. Apart from the changes to the drogue-cloud repository, I also needed to make changes in the [Helm charts repository](https://github.com/drogue-iot/drogue-cloud-helm-charts). You can find my PR over [here](https://github.com/drogue-iot/drogue-cloud-helm-charts/pull/1)

### Step 4: Tests & Documentation

Although at the time of writing, the tests don't work due to an unknown issue in the test suite itself, the PR submitted to the [drogue-cloud-testing](https://github.com/drogue-iot/drogue-cloud-testing) repository is [over here](https://github.com/drogue-iot/drogue-cloud-testing/pull/2).

The PR for making errors more informative, adding more comments, and adding documentation on expected behaviour is [over here](https://github.com/drogue-iot/drogue-cloud/pull/117),

## Final Outcome & State of the project

The endpoint is fully functional in all aspects except for encryption. Devices can publish data, as well as receive commands via the response. All my goals, except for the optional goal to support encryption, have been completed.

> Note on support for encryption: Currently, neither the base `coap-rs` library nor `Drogue Cloud` has support for DTLS. When they are implemented in the foreseeable future, the endpoint will be accordingly updated. There are also failures in the test suite that should be addressed soon.

## Epilogue

Google Summer of Code allowed me to explore and learn my way, at my pace. The lessons I received and the bonds made during my journey are things I'm going to carry with me for the rest of my life. I hope I get to have more experiences like this program in future. Thanks for reading my post!
