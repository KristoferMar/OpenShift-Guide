# Customizing Source-to-Image Builds

The programming language detection feature relies on finding specific file names at the root of the Git repository.

The S2I build process involves three fundamental components, which are combined to create a final container image

## Application source code
This is the source code for the application

## S2I
S2I scripts are a set of scripts that the RHOCP build process executes to customize the S2I builder image. S2I scripts can be written in any programming language, as long as the scripts are executable inside the S2I builder image.
- These scripts typically goes into the fllowing directory for storage
<pre>
.s2i/bin
</pre>

## The S2I builder image
This is a container image that contains the required runtime environment for the application. It typically contains compilers, interpreters, scripts, and other dependencies that are needed to run the application.

The S2I build process relies on some S2I scripts, which it executes at various stages of the build workflow. The scripts are as follows 

### assemble
- mandatory: yes
- description: The assemble script builds the application from source and places it into appropriate directories inside the image.

### run
- mandatory: yes
- description: The run script executes your application. It is recommended to use the exec command when running any container processes in your run script. This ensures signal propagation and graceful shutdown of any process launched by the run script.

### save-artifacts
- mandatory: no
- description: The save-artifacts script gathers all dependencies required for the application and saves them to a tar file to speed up subsequent builds. For example, for Ruby, gems installed by Bundler, or for Java, .m2 contents. This means that the build does not have to redownload these contents, improving build time. These dependencies are gathered into a tar file and streamed to the standard output.

### usage 
- mandatory: no
- description: The usage script provides a description of how to properly use your image.

### test/run
- mandatory: no
- description: The test/run script allows you to create a simple process to verify if the image is working correctly.
