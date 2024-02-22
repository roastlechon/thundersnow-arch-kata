# Thundersnow's O'Reilly Architectural Kata 2024

# Context / Problem Statement<a id="context--problem-statement"></a>

StayHealthy, Inc. is now expanding into the medical monitoring market, and is in need of a new medical patient monitoring system for hospitals that monitors a patient’s vital signs using proprietary medical monitoring devices built by StayHealthy, Inc.

https\://on24static.akamaized.net/event/44/64/40/0/rt/1/documents/resourceList1707409600669/privatekatakickoff1707409600669.pdf


## Current State<a id="current-state"></a>

_Describe the current system, application, or solution. State features and facts of the existing solution as it relates to the problems._

- Solution currently does not exist and is to be built


## Future / Desired State<a id="future--desired-state"></a>

_Describe the future state of the solution as features and facts that would be resolved by implementing the solution_

- Able to monitor patients via vital devices

- Able to send alerts to medical professionals

- Ability to customize alert thresholds based on configurable rules by professionals

- Ability to support upwards of 500 patients

- System will be managed on-premise with ability to be deployed to other hospitals


# Goals<a id="goals"></a>

_State goals or KPIs of desirable and measurable outcomes of the project._

- Highly available solution

- Easily deployable to other on-premise locations in the future

- Ease of adding new devices

- Scalable solution


# Requirements

## Functional Requirements

### FR1: Patient monitoring device transmits vital sign readings:

#### Heart rate: every 500ms

#### Blood pressure: every hour

#### Oxygen level: every 5 seconds

#### Blood sugar: every 2 minutes

#### Respiration: every second

#### ECG: every second

#### Body temperature: every 5 minutes

#### Sleep status: every 2 minutes

### Monitoring screen displays each patient vitals, rotating each patient every 5 seconds

### The software should analyze each patient’s vital signs, and alert when an issue is detected (e.g., decrease in oxygen level)

### Medical professionals should receive push alerts for potential problems, sent to their phone via a mobile app

### Alerts for potential problems should be pushed to the monitoring screens

### Medical professionals should be able to view vital sign history


### Medical professionals should be able to filter on time range


### Medical professionals should be able to filter on vital signs

### Medical professionals should be able to set alert thresholds for each vital sign

### System should be able to understand situational thresholds based on other vitals (sleeping patient can have blood pressure drop)

### Medical professionals should be able to receive alerts if vital sign is reached

### Medical staff should be able to generate holistic snapshots from a patient's consolidated vital signs at any time.

### Medical staff should be able to upload patient snapshot using HTTPS to MyMedicalData

## Non-Functional Requirements

### Availability

#### Must be available all the time to deliver vitals, alerts to monitoring devices

#### Device must send heartbeat to MonitorMe regardless of vitals.

### Performance

#### Each vital monitoring function must complete as fast as possible.



#### MonitorMe must deliver data to nursing stations with an average response time of 1 second or less



### Reliability

#### If any vital sign device (or software) fails, MonitorMe must still function for other vital sign monitoring (monitor, record, analyze, and alert).



### Security

#### MonitorMe does not need to meet HIPAA



#### MonitorMe needs to comply with standard security practices.



### Deployability

#### MonitorMe needs to be deployed to multiple hospital locations



#### MonitorMe (the entire system) needs to be an on-premise solution only



### Elasticity

#### Each patient monitoring device transmits vital sign readings at a different rate. Every half second to every hour



#### The system needs to be able to adjust from periods of lower alert analysis (a patient sleeping), to periods of greater alert analysis (a patient is awake).



### Scalability

#### A MonitorMe instance needs to support a max of 500 patients (in a hospital)



#### A MonitorMe nursing station can have 20 patients



#### MonitorMe needs to be able to expand to more vital sign monitoring devices

### Accuracy

#### MonitorMe needs to be as accurate as possible

#### Doctors must be able to get an accurate snapshot, regardless of which station the patient is currently registered to.

#### A patient needs to be able to move between MonitorMe stations which still ensures data accuracy.

# System Context


### Systems

* Monitoring Station App (per Nurses Station)
  * Interface with MonitorMe Software
  * Receive Push notifications from event brokerStayHealthy Mobile Application (per Medical Professional)
  * Interface with MonitorMe Software
* Receive Push notifications from event broker
* MonitorMe Software (on premise system)
  * Event Driven Broker(s)
  * Push Notifications
  * Vital Sign Alerting
  * Software Failure Alerting
* Database
  * Record and maintain patient data (24 hrs)
  * Record patient, device, push tokens, alert rules
* MyMedicalData
  * System which receives snapshots of patient data

### Actors

* Medical Professionals
* Nurses
* 8-Patient Monitoring Equipment

# IcePanel (C4 Model)

[IcePanel interactive diagram](https://s.icepanel.io/KyPu9LD0iOUJjY/9YPp)

![System Context](./Monitor%20Me%20-%20Arch%20Kata%20(Latest).png?raw=true" "System Context")
![Container Diagram](./MonitorMe%20-%20App%20Diagram%20(Latest).png?raw=true" "System Context")

[Journey Diagrams](./MonitorMe%20-%20C4%20Overview.pdf?raw=true)

  # Customer Journey

  ## Case 1: Nurse onboards patient with devices

  
  - Patient is assigned room in nursing station from nurse

  - Nurse collects patient information

  - Nurse prepares devices for patient

  - Nurse registers patient with devices on the Monitoring Station App

  - Monitoring Station App sends patient and device registration details to MonitorMe

  ## Case 2: Nurse assigns medical professional for alerting purposes

  
  - Nurse selects medical professional in system to be alerted

  - Medical professional logs into App and accepts patient to monitor

  - App sends push token to MonitorMe to register push subscription

  ## Case 3: Medical Professional and/or nurse configures alerts

  
  - Nurse selects patient is selected either through the MonitorMe station or the StayHealthy app

  - Nurse selects alert rules for vital signs

  - Nurse creates alert rules if none existing

    - Rules regarding thresholds for each vital sign can be configured, or defaults may be used

    - Thresholds can be configured conditionally, dependent on factors such as whether a patient is awake or sleeping

  - App sends request to create alert rules

  - App sends request to associate alert rules with patient

  ## Case 4: MonitorMe device(s) monitor patient

  
  - Monitoring device sends vital data to MonitorMe Event Broker

  - Asynchronously


    - MonitorMe Event Broker sends vital data to Raw Vitals Service

    - MonitorMe Event Broker sends vital data to Alert Service to evaluate vitals

    - MonitorMe Event Broker streams vital data to Vital Events Service

  - Raw Vitals writes to MonitorMe Raw Vital Database

  ## Case 5: Nurse monitors monitoring station for vitals and alerts

  
  - Alerts are sent to the monitoring station

    - Vitals are pushed from the event broker to the alerts service

    - If the vitals trigger an alert, the alert is pushed to the platform push system

    - The platform push system pushes the alert to the monitoring station

  - Vitals are pushed to the monitoring station from the vital events service

    - A vital event is sent from the device to the event broker

    - The event broker pushes the vital event to the vitals service

    - The vitals service pushes the vital event to the platform push system

    - The platform push system pushes the vital event to the monitoring station

  ## Case 6: Nurse transfers patient to another station

  
  - The nurse finds the patient in the current monitoring station

  - The nurse unregisters the patient from the current monitoring station

    - The nurse submits a request to the devices service to remove the current monitoring station’s association with the patient

    - Vitals data is still recorded for the patient; that data is simply not pushed to a monitoring station until the new station has been setup

  - The nurse registers the patient with the new monitoring station

    - At the new monitoring station, the nurse finds the patient to be transferred

    - A request is made to register the new station with the patient

  ## Case 7: Nurse replaces device

  
  - The nurse finds active patient in Monitoring Station App

  - The nurse changes the active device registered to that patient to the new device

  - Monitoring Station App sends request to Gateway to change device

  - Gateway sends request to Devices to change patient device from old to new

  - Devices sends update to database

  - Devices responds success

  ## Case 8: Nurse deboards patient

  
  - The nurse removes the vital sensors from patient

  - The nurse finds patient in Monitoring Station App

  - The nurse closes out the patient

  - Monitoring Station App sends request to broker to remove patient from Active list in MonitorMe

  ## Case 9: Medical Professional and/or nurse pulls snapshot of patient vitals

  
  - Nurse requests snapshot in monitoring station app

  - Monitoring station app sends snapshot data to Snapshots Service

  - Snapshots Service saves event in MonitorMe Database

  - Snapshots Service sends snapshot to MyMedicalData

  ## Case 10: Medical Professional and/or nurse receive alert(s)

  
  - One or more vital sign devices measures a value

  - The value is pushed to the MonitorMe event broker

  - The value is pushed from the event broker to the alert service

  - The alerts service evaluates that the value has exceeded a threshold

  - The alerts service sends an alert to the Platform Push System

  - The Platform Push System notifies the apps and stations associated the patient

  # Architectural Decision Records

  ## ADR1: Use of event driven architecture to deliver notifications

  ### **Status:**
  Proposed
  ### **Context:**
  Need to deliver push notifications to medical professionals and alerts to monitoring screens in a timely manner
  ### **Relates to:**
  * NFR4

  * NFR14
  ### **Decision:** 

  To address the requirement of timely delivery and scalability, the decision is to adopt an event-driven architecture for delivering notifications.The decision is based on the recognition that event-driven architecture offers several advantages in terms of real-time processing, scalability, and decoupling of components. By leveraging this architecture, the system can react to events, such as patient updates or critical alerts, in real-time and push relevant notifications to medical professionals and monitoring screens without delay.

  ### **Consequences:**

  - Real-time responsiveness: Pushing notifications ensures immediate delivery to recipients, enabling prompt response to critical events.

  - Reduced latency: By eliminating the need for recipients to poll for updates, push notifications minimize latency and ensure timely dissemination of information.

  - Efficient resource utilization: Pushing notifications conserves resources by delivering updates only when necessary, reducing unnecessary polling requests.* Potential for message loss: In scenarios of high message throughput, there is a risk of message loss or delivery failures, necessitating robust error handling and retry mechanisms.**

  ## ADR2: Use of microservices and API Gateway

  ### **Status:**
  Proposed
  ### **Context:**
  In order to isolate raw vital delivery from general crud APIs, we need to build out separate microservices with API’s exposed. APIs have differing needs where some need to be highly available, and others are not as important due to infrequent use.
  ### **Relates to:**
  * NFR3

  * NFR4

  * NFR5

  * NFR14

  ### **Decision:** 

  We chose to use microservices that are exposed via an API Gateway. Microservice pattern allows us to have disparate business logic and responsibilities for each service. It is also a choice for us to not create separate databases for each vertical (microservice) to reduce cross communication and complexity.

  ### **Consequences:**

  * Separation of concerns: individual feature requirements can be done independently from each other

  * Extensibility: versus a monolithic application, microservices allow MonitorMe to be extended to support any future possible new feature requirements with minimal refactoring or re-architecting

  * Reliability: if a vital sign device fails, the microservices can still function

  * Performance: individual monitoring functions can be used concurrently/in-parallel to avoid blocking one another (when possible), allowing each function to complete as quickly as possible

  * Complexity: development, testing, and deployment are not as straightforward as a monolithic application

  * Not creating databases for each microservice reduces overhead and complexity. We mitigate performance issues by our below decision of using a separate database for vital data and other non critical data.**

  ## ADR3: Methodology of ingesting raw vital data from MonitorMe devices

  ### **Status:**
  Proposed
  ### **Context:**
  Solution needs to support 500 \* 8 metrics (and be flexible for more) every half second.

  ### **Relates to:**
  * NFR9

  * NFR10

  * NFR11

  * NFR13
  ### **Decision:** 

  Use an Event Broker to send vital data to a raw vitals service that writes to the raw vitals database table.

  ### **Consequences:**
  * Leveraging an event broker should be able to efficiently handle the large volume of incoming data at different cadences especially if the different vitals are subscribed to different topics**

  ## ADR4: Choice of database technology to store data

  ### **Status:**
  Proposed
  ### **Context:**
  Raw vital data records can be stored upwards of a count of 700 million. Database needs to support 700 million records in 24 hours. We also need to store disparate data to relate back to the device raw data. There is also a read/write concern whereby one service could be responsible for writing to the database, and other services could be responsible for reading only.Note: 500 \* 8 \* 2 \* 60 \* 60 \* 24 = 691,200,000 (2 comes from half a second for heart rate vitals)
  ### **Relates to:**
  * NFR6
  ### **Decision:** 

  We will use a Postgres database with read / write replicas to allow for high availability and scalability.

  ### **Consequences:**
  * Scalability: Utilizing Postgres with read/write replicas enables horizontal scaling to accommodate the influx of data and distribute read operations across multiple instances, enhancing system performance.

  * High Availability: By employing replicas, the system gains fault tolerance and resilience against failures, ensuring continuous availability of data even in the event of hardware or network issues.

  * Data Integrity: Postgres's ACID compliance guarantees data integrity, maintaining consistency and reliability of stored information critical for healthcare applications.

  * Maintenance Overhead: Managing read/write replicas and ensuring synchronization between them may introduce additional maintenance overhead, requiring careful monitoring and management to maintain system stability and performance.

  * Extensive Data Types: PostgreSQL supports a wide range of data types including numeric, text, JSON, XML, spatial, and custom types, enabling the storage and manipulation of diverse data formats commonly found in healthcare applications, ensuring flexibility and versatility in data management.**

  ## ADR5: Methodology of onboarding patients to differentiate device data at any given point in time

  ### **Status:**
  Proposed
  ### **Context:**
  Solution needs to support related device data, which would not send patient data along with it, to patient data that is defined by the nurse/professional. This means that device data can be kept raw and stored uncorrelated to patients. Patient data can live separately and related via a join table.
  ### **Relates to:**
  * NFR6

  * NFR12

  * NFR16
  ### **Decision:** 

  Implement a database schema that separates device data from patient data and includes a join table for linking them when necessary.

  ### **Consequences:**
  * Improved Data Privacy: By keeping device data and patient data separate, the risk of unintentional exposure of patient information is minimized.

  * Enhanced Data Management: Healthcare professionals can efficiently manage and analyze device data without being encumbered by patient data, leading to better decision-making.

  * Scalability: The proposed methodology allows for scalability as it accommodates a growing volume of device data and patient records without compromising system performance.

  * Flexibility: Separating device data from patient data provides flexibility in adapting to changes in data sources or patient management protocols, as well as allowing patients to be registered to any MonitorMe nursing station**

  ## ADR6: Choice of how the nursing station shows data

  ### **Status:**
  Proposed
  ### **Context:**
  Nursing stations could be pushed to monitor vital data or can retrieve/poll monitored vital data periodically. Pushing data directly can cause performance and battery issues on mobile apps. Alerts, however, are important and should be pushed directly to the devices. When it comes to monitoring, we should honor the various rates in which each device is sent to MonitorMe to preserve accuracy.
  ### **Relates to:**
  * NFR1

  * NFR4
  ### **Decision:** 

  In order to not transmit large amounts of data from the system, we can leverage Server Sent Events so that data can be transmitted on request. When the mobile app is inactive, data is not pushed to the device, and when the mobile app comes back to foreground, data can be sent to the device. This allows for data to be up to date and accurate, while also being efficient. The same applies to the monitoring stations, although they are connected far more often than the mobile app. Frontend logic can be applied to ensure the server sent events are being sent and to restart the connection if necessary.

  ### **Consequences:**
  * Data is sent only when client applications are ready to receive the data

  * Client applications do not need to manage pulling the data regularly at the required intervals - the backend system manages that

  ## ADR7: Methodology of configuring alert thresholds

  ### **Status:**
  Proposed
  ### **Context:**
  Patients can have different alert thresholds that could be configured differently.
   We need a way to allow medical professionals to create alert rules and assign them to patients. This allows raw vitals to be sent to an alert system to be evaluated and delivered to end users.
  ### **Relates to:**
  * FR9
  ### **Decision:** 

  Implement a flexible alert threshold configuration module within the MonitorMe system that allows medical professionals to define and customize alert rules based on individual patient needs. This module should include features such as medical parameter threshold setting threshold adjustment and contextual threshold alerts.
  
  ### **Consequences:**

  * Custom Patient Monitoring: Customizing alert threshold to individual patient conditions help in early detection of abnormalities. Especially the ability to configure alerts on a patient by patient basis allows for the catering of unique needs and conditions of each individual.

  * Conditional/Contextual Monitoring: Contextual alerts help in the reduction of unnecessary or false alerts.


  ## ADR8: Sending snapshots to MyMedicalData

  ### **Status:**
  Proposed
  ### **Context:**
  Patients can wear multiple devices at any given point in time (eg. ICU to CCU to general)
  ### **Relates to:**
  - NFR4* NFR14

  * NFR15
  ### **Decision:** 

  Sending data to MyMedicalData should be done as a snapshot of the date range of the raw vital data. By letting the microservice handle querying data given a patient's usage of devices at a point in time, we can stitch together multiple device usages by a patient if a patient were to move from different nursing stations. 
  ### **Consequences:**
  * Improved Data Privacy: By ensuring that each patient's data is segregated from another, we can guarantee that the data sent in the snapshot is for the correct patient, regardless of device.

  * Increased Flexibility: By separating patient data from the device data, we can ensure that regardless of what device or monitoring station the patient starts at that we can always deliver an accurate snapshot of their metrics during the stay.**

  ## ADR9: Choice of technology for frontends

  ### **Status:**
  Proposed
  ### **Context:**
  The Nurse Monitoring Station and StayHealthy mobile app are consumers of the MonitorMe system.
   The mobile app has the ability to receive alerts in the same methodology that the monitoring station does, and therefore could leverage the same benefits in using shared codebase/technologies.
  ### **Relates to:**
  * FR2

  * FR4
  ### **Decision:** 

  Our approach for the frontends is to use React Native.
   This allows us to use a shareable design system across mobile apps and any other platform that the nursing monitoring station uses (assumed to be Windows). Code can also be written in one language and written per platform.

   ### **Consequences:**
   * Possibly lose out on native platform specific performance enhancements

  * Vendor lock in to the React Native framework**

  ## ADR10: Choice of backend microservice language

  ### **Status:**
  Proposed
  ### **Context:**
  The backend code needs to be reliable, fast, scalable, and elastic.
  ### **Relates to:**
  * NFR3

  * NFR4

  * NFR10

  * NFR11

  * NFR12
  ### **Decision:** 

  Our approach for the backend microservices is to use Go. It is a highly-performant, strongly-typed, efficient language. This allows for quick execution, and quick deployments. Go allows us to make full use of the available hardware, in a CPU and memory-efficient manner.

  ### **Consequences:**

  * The microservices can perform their functions very quickly, with code processing time minimized greatly

  * Go scales well vertically

  * Hardware permitting, Go can handle quick bursts of increased processing

  * Memory is managed by a garbage collector, rather than explicitly managed by the developer, which can introduce periodic spikes in latency**

  ## ADR11: Deployment strategy of on-premise system

  ### **Status:**
  
  Proposed
  ### **Context:** 

  Given the need to have high availability on-premise with microservices architecture, we need an approach to deploy services and scale as needed.
  
  ### **Relates to:**
  * NFR-3

  * NFR-4

  * NFR-14

  * NFR-15

  * NFR-16

  ### **Decision:** 

  Our deployment approach is to use an on-premise Kubernetes solution with multiple nodes to provide redundancy and high availability. This allows for services to scale per usage and and failover to other nodes in case of hardware failures.

  ### **Consequences:**

  * High availability: Kubernetes can have multiple nodes which allow deployments in a variety of physical host infrastructure. If one service fails due to a host failure, it can be quickly stood up in another physical host (node).

  * High scalability: Building the services in a containerized/stateless manner allows us to scale horizontally to support additional vital collection devices. Containerized services can be built in any language

  * Complexity in maintenance: Using kubernetes has abstractions on deployments which require highly skilled DevOps.


  # Concerns / Assumptions

  * For cases where patients stay longer than a day, should we care about storing that data long term besides the use of manual snapshots

  * Monitoring of vitals within 24 hour period is specifically about the raw data set, and not snapshots

  * Snapshots could be stored indefinitely

  * Snapshots information requirement is in the monitoring station app.

  * Assumption that the MonitorMe device can send the value and unit appropriately based on the specific vital its recordings

  * In cases where the patient moves from one monitoring station to another, we need to ensure that the patient's data is accurate and available for snapshot, regardless of monitoring station.

