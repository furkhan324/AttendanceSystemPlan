# AttendanceSystemPlan

## Synopsis

High-level design for attendance system. Following section details user-interactions possible, pseudocode implementations of the most relevant functions that power the user-interactions, and lastly any relevant APIs.

<h3>Requirements/Contingencies:</h3>

- For teachers: UI to set class times/show attendance (Web only)
- For students: Manual UI to enroll/check-in (Mobile App/Web) 
- For students: Push Notifications/Geolocation for auto-check-in(Mobile)
- For both:

  - Firebase Authentication to map user information such as SID, Name, Other student info - Phone #/Email to user account
  - Node/Express server to serve READ/WRITE requests to MongoDB from mobile & web (Deployed on Heroku)
    - Heroku has a convenient MongoLabs plugin that enables easy MongoDB integration
  - Mongo Database as main noSQL storage solution for all application data
  - (Auto-check-in) Node server to host time-based webhooks that send out push notifications for auto-check-in
    - APNS (Apple Push Notification Server) for iOS push nots
    - GCM (Google Cloud Messenger) for Android push nots
  
## Platforms

- Web for Administrators & Students
- Android/iOS app for (Students only)
  - Saves trouble of implementing less mobile-friendly admin/teacher features for app version

##Implementation/User Flow:

- <h3>Onboarding/Enrollment</h3>
  - For teachers & Students: OAuth using email , password using <b>Firebase</b> --> Firebase provides necessary security suite for OAuth(Hashing, Salting, etc.)
    to save time from custom implementation. 
  - Write User objects to DB after OAuth (if sign-up)
  
    ``` JavaScript 
    //Example Teacher/Student Schemas 
    //userObject
    var teacher1 = {userid:1, //unique per user
      accountType : "Teacher",
      email: "teacher@berkeley.edu",
      classesTaught : [] //classObjects to be implemented later
    }
    var student1 = {userid:2, //unique per user
      accountType : "Student",
      email: "student@berkeley.edu",
      classesTaken : [] //classObjects to be implemented later
    }
    
    Users.add(teacher1, student1);
    ```
  
  - For Teachers : Present calendar/scheduler interface. Neat jQuery plugin for this is  @ <a>https://github.com/telerik/kendo-ui-core</a>.
    - Angular.js on front-end for instructor to submit class. <a>https://angularjs.org/</a> 
    
    <code>ng-submit</code>
    
    ```JavaScript
    //Example classObject
    //classObject
    
    var class1 = {classid:1, //unique per class
      teacher: "Teacher Name",
      className : "Class Name",
      classTimeAndDay : dayAndTimeOfClass, 
      classLocation = geocodeAddress("Address of classroom"), //returns an object with Latitude and Longitude attributes
      students = [], //studentObjects enrolled in class
      generateAttendanceCode = function {
      return uniqueAttendanceCodePerClassSession; //function when called returns a unique attendance code per session of class, to be used later
      classAttendancePerSession = [[], [],....]  //array of array of userObjects who attend indexed session
      latestSessionCode = "sessionCode" //latestSessionCode generated
    }
    
    Classes.add(class1)
    
    }
    ```
    
  - For students: Instructors provide classid on first day of class to enroll. OR If the student is near/in the class, geoQuery returns/UI presents list of all classes at that place at that time available for enrollment.
  
    ```JavaScript
    func onCLASSIDInput(UserObject currentStudent, String classid){
    
      ClassObject classOfInterest = Classes.findByCLASSID(classid)
      currentStudent.classesTaken.append(classOfInterest)
      classOfInterest.students.append(currentStudent)
      
      //Equivalent code written in Swift/Java for Android/iOS
    }

    ```
- <h3> Check In (Manual)</h3>
  - Suitable if location-services is turned off or students want to check-in from computer(web)/phone unavailability
  - For teacher: Application Runs generateAttendanceCode() on currentClassObject and generates a unique attendance code for that session of the class.
  Teacher posts code on board/projector.
  
    <code>currentClass.latestSessionCode = currentClass.generateAttendanceCode()
  //currentClass is classObject (see schema above)
    </code>
  
  - For students: Student manually inputs code given by teacher. Application checks to see if it matches with the latestSessionCode of the class.
  
    ```JavaScript
  func onSessionCodeInput(String sessionCodeInput, UserObject currentStudent, ClassObject classOfInterest, int sessionNumber){
    if (sessionCodeInut == classOfInterest.sessionCode){
    
      classOfInterest.classAttendancePerSession[sessionNumber].append(currentStudent)
      print("Attendance was logged succesfully")
    }else
    
      print("Incorrect Code + Attendance was not logged")
  }
  //Equivalent code written in Swift/Java for Android/iOS

    ```
- <h3> Check In (Auto-check-in) - (Mobile Only) </h3>
  - Time based Webhooks on Node server send out push notifications for all studentObjects in the .students attribute of the classObject upon .timeAndDay 
  - Push Notifications routed to devices using GCM, APNS respectively
  - Push Notification carries the following payload
  
    ```JavaScript
  var payload = {
    sessionCode = currentClass.currentSessionCode, //payload carries currentClassSessionCode
    classLocation = {latitude, longtitude}
  }
    ```
  
  - For students: Student taps on push notification. Following function runs on app using the information provided by the payload of the push-notification.
    ```JavaScript
    
    func executeUponOpenOfPushNotification(Payload pl){
      var currentStudentLocation = getCurrentStudentLocation() //pretty obvious
    
      if ( currentStudentLocation.isInCloseProximityOf( pl.classLocation ) )
        onSessionCodeInput(pl.sessionCode, ...) //onSessionCodeInput() as defined above logs attendance
      }else{
        print("Not in class so could not log attendance")
      }
      //Equivalent implementation in Swift or Java run when push notification opened
    ```
    
  - Function run basically checks to see if the student is in the class. If yes, then do DB Posts to mark student as attended. 
  If not alert user.
  
- <h3>Admin Panel (Web)</h3>
  - For teacher: Application parses classObject data to see attendance. 
  - Attendance Trend Visualization: Flot.js is a neat library that can be used to render attendance trends in graph format
  - Important metrics such as dailyAttendance, dailyAbsences, averages, etc. can be determined with relevant operations on .students and .classAttendancePerSession attributes of classObject
  
## Contributors

Mohammed Abdulwahhab (@furkhan324)

## License

Code may not be copied, edited, or reproduced in any form without the consent of the contributors.



