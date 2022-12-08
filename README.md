# CICD_Java_gradle_application

This application is java spring boot web application  

build tool is ** gradle **

when we build the code using command ```./gradlew build ``` it will generate war file. that war can be placed in tomcat server to see application web page

code is integrated with sonarqube plugin which help us in static code analysis 

``` ./gradlew sonarqube ```


# jenkins Issues
1. Gradle Sonarqube issue (Remove Docker from stages, we do not docker image for openjdk)
2. Quality Gate Json issue (check sonar server in manage jenkins and make sure it is http://sonar:9000   -- remove "/")
