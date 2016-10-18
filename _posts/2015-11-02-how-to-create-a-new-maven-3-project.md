I like to use [Maven](https://maven.apache.org/) for my projects. I don’t think that there is any developer team not using a build tool today and there is also no reason to argue why you SHOULD use a build tool to manage your dependencies. I have read nice things about [Gradle](https://gradle.org/) as well on the Internet and I will definitely check it as soon as I have time.

So, as with any tool you use, there will always be these commands you do not frequently use and when you have to, you don’t always remember the exact syntax. And you have to check your machine’s history. Or google.

For some reason I never found it trivial to create a brand-new project in maven, so I put the command here, basically for my own future references, but if someone finds it helpful I will be really glad!

    mvn archetype:generate -DgroupId=<group-id> -DartifactId=<artifact-id> -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

where <group-id> and <artifact-id> are the ones desired for the project.
