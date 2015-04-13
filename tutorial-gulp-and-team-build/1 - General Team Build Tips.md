#Building Cordova Apps in a Team / Continuous Integration Environment
With the release of Visual Studio 2015, you now have a number of options for how you can integrate Cordova apps with your favorite team / continous integration (CI) server thanks to the fact that projects created in Visual Studio are standard [Cordova Command Line Interface](http://cordova.apache.org/docs/en/edge/guide_cli_index.md.html#The%20Command-Line%20Interface) (CLI) projects. In this tutorial, we will cover a few different approaches for building Cordova projects outside of Visual Studio.

For specific abridged tutorials for specific build systems, you may find this sample **[team-team-build node module](http://aka.ms/tacoteambuild)** useful along with the following tutorials:

-   [Gulp - Using Gulp to Build Cordova in a Team / CI Environment](http://aka.ms/cordovagulp#ci)
-   [Team Foundation Services 2015 and Visual Studio Online](http://aka.ms/cordovatfs2015)
-   [Team Foundation Services 2013](http://aka.ms/cordovatfs2013)

This article will go through the general approach for tackling a number challenges that exist when building Cordova apps and cover what the taco-team-build node module effectivley does behind the scenes.

**Note that Team Foundation Services 2013 cannot easily take advantage of the workflow described here (though 2015 can) as it is MSBuild based. See the [Team Foundation Services 2013](http://aka.ms/cordovatfs2013) tutorial for details.**


##Basic Workflow
Each build server technology is a bit different and in this article we will focus on the general steps required to build a Cordova app regardless of technology using the Cordova Command Line Interface.

The basic flow for building a Cordova app is simple on the surface:

1.  Check the project out from source control

2.  Add the platforms you want to build to the project using the “cordova platform add” command. Ex:

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	cordova platform add android
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

3.  Build the project using the “cordova build” command:

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    cordova build android --release
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Cordova CLI is node.js based, so these exact same steps can be run from an OSX machine or from a cloud hosted VM like [MacInCloud](http://www.macincloud.com/) or from Windows. See the [Cordova CLI documentation](http://cordova.apache.org/docs/en/edge/guide_cli_index.md.html#The%20Command-Line%20Interface) for additional details.

Exactly how these steps are executed will vary depending on your build server. However, there are a few challenges that may not be immediately obvious when setting up an automated build environment. This are article will describe some techniques for dealing with these common problems.

##Installing Dependencies
Cordova builds require that a number of dependencies be properly installed and configured on the system. However, exactly which dependencies are required varies based on the Cordova “platform” (Android, iOS, Windows 8.0/8.1 and Phone 8.1, Windows Phone 8.0) you want to build.

Installing Visual Studio 2015 with the Tools for Apache Cordova option will automatically install these dependencies but you will still need to configure some of the environment variables by hand for Android. See [Team Foundation Services 2015 and Visual Studio Online](http://aka.ms/cordovatfs2015) for a summary of these variables.

Otherwise you can manually install only those dependencies that are needed for building the platforms you are interested in.

1.  Install Node.js and make sure it is available to the system user you intend to have run your builds

2.  Install the platform specific dependencies on the server and make them available to this same user. See the following guides for details:

    -   [Android Platform Guide](http://cordova.apache.org/docs/en/edge/guide_platforms_android_index.md.html#Android%20Platform%20Guide)

        1.  You do not need to install Android Studio or Eclipse

        2.  Instead you may download and install one of the [“SDK Tools Only” packages](http://developer.android.com/sdk/index.html#Other).

        3.  When building, you encounter an error telling you that you need to install specific SDK versions or tools depending on the version of Cordova you are using.
            a.  You can install these using [the Android SDK Manager](http://developer.android.com/tools/help/sdk-manager.html).
            b.  The only thing for a specific SDK required to build is the “SDK Platform” so you may uncheck other options. Android system images in particular are large and are not needed.
            c.  Be sure to also install the “platform tools”.
            d.  Projects created using Visual Studio will typically use either the SDK from API 19 (4.4.2) or API 21 (5.0.1).

    -   OSX only: [iOS Platform Guide](http://cordova.apache.org/docs/en/edge/guide_platforms_ios_index.md.html#iOS%20Platform%20Guide)

    -   Windows only:
        1.  [Windows and Windows Phone 8.1+ Platfrom Guide](http://cordova.apache.org/docs/en/edge/guide_platforms_win8_index.md.html#Windows%20Platform%20Guide)
        2.  [Windows Phone 8.0 Platform Guide](http://cordova.apache.org/docs/en/edge/guide_platforms_wp8_index.md.html#Windows%20Phone%208%20Platform%20Guide)

###A Note on TypeScript
Unlike Visual Studio, it’s important to note that the base Cordova CLI does not itself automatically compile TypeScript code. If you are using a build language like Gulp or Grunt, there are convenient plugins that you can use to compile your TypeScript code. Otherwise there is also a node.js based command line utility that works both on Windows and OSX. See the following links for additional details:

-   [Compiling TypeScript from the command line](http://www.typescriptlang.org/Tutorial)
-   [gulp-typescript](https://www.npmjs.com/package/gulp-typescript)
-   [grunt-typescript](https://www.npmjs.com/package/grunt-typescript)

##Cordova Challenges
When building Cordova projects in a server environment, there are a number of challenges you may encounter. This tutorial will describe simple ways to handle these problems without going into specifics on particular CI servers so that this information can be adapted to your favorite build technology.

If you are looking for a quick solution you may want to check out the [Gulp](http://aka.ms/cordovagulp) tutorial and this [GitHub repo](http://aka.ms/tacoteambuild) with the sample taco-team-build node module designed to help resolve these problems regardless of build system.

1.  **Building with Multiple Versions of the Cordova CLI.** While in an ideal world everyone would use the edge version of the Cordova CLI and associated platforms, the reality is that for a given build server you will want to use multiple versions of the Cordova CLI. This means that the common practice of installing Cordova [globally](http://blog.nodejs.org/2011/03/23/npm-1-0-global-vs-local-installation/) will not work.

2.  **Adding Cordova Platforms.** As of Cordova 4.2.0, the “cordova platform add” CLI command still needs to be explicitly fired unless you opted to check in the contents of the “platforms” folder which is not recommended.

3.  **Generating an iOS App Store Package.** By default, the iOS build in Cordova generates an app rather than an “.ipa” for device deployment. An ipa can be generated from the command line quite easily but build servers like Jenkins require an extra step.

4.  **Visual Studio Specific Features**
	1.  **Supporting res/native.** The “res/native” folder in Visual Studio projects provides a useful way to update native project build artifacts without having to check in the platforms folder. It is not currently a base Cordova CLI feature.
    2.  **Supporting the Task Explorer.** Visual Studio now has a convenient Task Explorer that allows you to attach Gulp tasks to build events.

###Building with Multiple Versions of the Cordova CLI
The Cordova CLI is a standard node.js module and thus can be installed either [globally or locally](http://blog.nodejs.org/2011/03/23/npm-1-0-global-vs-local-installation/). The trick, then, is to use a local installation of the Cordova CLI rather than a global one. There are two different methods that you can use to install Cordova locally: at the project level and in a global
cache.

#### Project Level
Installing and using the correct version of the Cordova CLI at the project level is simple thanks to something called [package.json](https://docs.npmjs.com/files/package.json). Here is the general approach:

1.  Create a package.json file in the root of your Cordova project.
2.  Add the following json to the file where “4.3.0” is the version of the Cordova CLI you intend to use:

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    {
    	"devDependencies": {
    		"cordova": "4.3.0"
	    }
    }
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

3.  Check this into source control with your project.

4.  Configure your build system to run the following command as its first task. This will then install the correct version of the CLI in a new “node\_modules” folder under your project.

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    npm install
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

5.  When executing a Cordova CLI command for your build task, you can then use the following:

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ./node_modules/cordova/bin/cordova
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    Ex:

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ./node_modules/cordova/bin/cordova platform add android
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The downside of this method is that you will end up installing the Cordova CLI each time you execute a “clean” build which will slow down your build times particularly on Windows as the CLI consists of around 25mb of content.

### Global Cache
To avoid re-installing each time, you can take advantage of Visual Studio’s **taco.json** file and a node.js script to do the installation in a specific location that you then use to execute Cordova commands.

1.  Add an environment variable to your system (or build) called **CORDOVA\_CACHE** pointing to where you a "cache" of the different versions of the Cordova CLI used to build your projects.

2.  Add this node.js script to your project and call it “setup-cordova.js”:

	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    var fs = require('fs'),
        path = require('path'),
        exec = require('child_process').exec;

    // Load taco.json 
    var taco = require("./taco.json"); 
    var cordovaVersion = taco["cordova-cli"];

    // Check if Cordova is already present in the cache, install it if not
    var cordovaModulePath = path.resolve(process.env["CORDOVA_CACHE"], cordovaVersion);
    if (!fs.existsSync(cordovaModulePath)) {
        fs.mkdirSync(cordovaModulePath);
        fs.mkdirSync(path.join(cordovaModulePath, "node_modules"));
        console.log("Installing Cordova " + cordovaVersion + ".");
    	exec("npm install cordova@" + cordovaVersion, { cwd: cordovaModulePath }, function (err, stdout, stderr) {
            console.log(stdout);
            if (stderr) {
                console.err(stderr);
            }
            if (err) {
                console.err(err);
                process.exit(1);
            }
    		console.log("Cordova " + cordovaVersion + " installed at " + cordovaModulePath);
        });
    } else {
    	console.log("Cordova " + cordovaVersion + " already installed at " + cordovaModulePath);
    }

    // Create shell scripts
    if (process.platform == "darwin") {
        // OSX
	    fs.writeFileSync("cordova.sh", "#!/bin/sh\n" + path.join(cordovaModulePath, "node_modules", "cordova", "bin", "cordova") + " $@", "utf8");
        fs.chmodSync("cordova.sh", "0777")
    } else {
        // Windows
    	fs.writeFileSync("cordova.cmd", "@" + path.join(cordovaModulePath, "node_modules", "cordova", "bin", "cordova") + " %*", "utf8");
    }
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


3.  In your team / CI build definition or script, add a build task to execute “node setup-cordova.js” from your project root

4.  Use “./cordova.sh” (OSX) or “cordova.cmd” (Windows) to run additional Cordova commands

Note that this same script can be easily adapted to a [Gulp build task](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md). See the [Gulp](<http://aka.ms/cordovagulp) tutorial and the samples [GitHub repo](http://aka.ms/tacoteambuild) for additional information.

###Adding Platforms
Adding platforms in Cordova is quite simple using the “cordova platform” command. Ex:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cordova platform add android
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

However, there are a couple of common problems when executing this command that you could run into.


1. **Platform Download Messages Result in Build Failures.** Where things can get a bit tricky is that node.js emits warnings to “Standard Error.”  The issue is that "platform add" command can result in warnings being reported when the CLI is downloading a version of a given Cordova platform for the first time. This is not an error, but some build systems will assume anything sent to standard error means a build failure occurred.

    Many CI systems provide a “continue on error” option that you can select to get around this particular problem or you can pipe standard error to standard out if you'd prefer.
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	cordova platform add ios 2>&1
	~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

2. **Errors During Incremental Builds.** If you are doing an incremental build and the platform you are build has already been added, the resulting exit code will be non-zero and may be interpreted as a build failure. If your build system supports a “continue on error” option for a given task, you can simply select that.

	However, a more robust solution is to simply conditionally call “platform add” if the appropriate folder in the platforms folder in your project is not found. In the scripts below replace “cordova” with the appropriate command from the “Building with Multiple Versions of the Cordova CLI” section above.

    Windows:

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    IF NOT EXIST platforms/android CALL cordova platform add android
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    OSX:

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    if [ ! -d "platforms/android" ]; then cordova platform add android; fi;
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

###Generating an iOS App Store Package
In order to distribute your iOS application you will need to generate an “iOS App Store Package” or “ipa” file. These files can be imported into iTunes or enterprise app stores in addition to being distributed to the Apple App Store via the [Application Loader](https://itunesconnect.apple.com/docs/UsingApplicationLoader.pdf).

#### Using xcrun
Future versions of the Cordova CLI will likely support generating these archives directly. However, current state it is not supported so an Xcode command line tool needs to be used instead. Ex:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
xcrun -v -sdk iphoneos PackageApplication source.app -o dest.ipa
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In Cordova projects, the source “.app” package can be found in the platforms/ios/build/device folder in your project after a successful Cordova “device” build. As an important detail, \<source\> and \<dest\> above should be **absolute paths** and the name of the package is taken from the “Display Name” (widget/\@name) in config.xml which may not match your project folder name. Ex:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
xcrun -v -sdk iphoneos PackageApplication "/Users/cdvusr/Documents/cordova/myapp/platforms/ios/build/device/My Cordova App.app" -o "/Users/cordova/Documents/cordova/myapp/platforms/ios/build/device/My Cordova App.ipa"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each build system has a different mechanisms in place for passing the absolute path of the project to shell scripts, but typically it involves the use of an environment variable. For example, in Jenkins you can use the following:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
xcrun -v -sdk iphoneos PackageApplication "${WORKSPACE}/platforms/ios/build/device/My Cordova App.app" -o "${WORKSPACE}/platforms/ios/build/device/My Cordova App.ipa"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This command will automatically match a developer a developer (when building Debug) or distribution (when building Release) signing identity based on the app’s package name. When you are using res/native, you can place a custom [build-debug.xcconfig ](https://github.com/apache/cordova-ios/blob/master/bin/templates/scripts/cordova/build-debug.xcconfig) or [build-release.xcconfig ](https://github.com/apache/cordova-ios/blob/master/bin/templates/scripts/cordova/build-release.xcconfig) file in res/native/ios/cordova in your Cordova project to override signing identities and other [build settings](https://developer.apple.com/library/ios/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/0-Introduction/introduction.html#//apple_ref/doc/uid/TP40003931-CH1-SW1) for these configurations.

However, additional command line arguments can also be passed such as “--sign” to resign the app using a specific signing identity or a path to a .p12 file and “--embed” to specify the path to a provisioning profile. Ex:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
xcrun -v -sdk iphoneos PackageApplication "${WORKSPACE}/platforms/ios/build/device/My Cordova App.app" -o "${WORKSPACE}/platforms/ios/build/device/My Cordova App.ipa" –-sign "/path/to/signing.p12" --embed "/path/to/some.mobileprovision"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### Solving Keychain Permission Errors
When auto-matching, you may hit permissions issues when using a build server like [Jenkins](http://jenkins-ci.org/) because the agent does not have permissions to access the login keychain. To solve this problem, you’ll need to unlock the keychain before you build and package your Cordova app.

Most build servers provide a way to inject secure environment variables before executing build tasks. In Jenkins this is accomplished by using the “Environment Injector Plugin.” By then setting a KEYCHAIN\_PWD environment variable you can add the following command to your build.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
security unlock-keychain -p ${KEYCHAIN_PWD} ${HOME}/Library/Keychains/login.keychain
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To then build and package your iOS app, you can run the following commands:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
security unlock-keychain -p ${KEYCHAIN_PWD} ${HOME}/Library/Keychains/login.keychain
cordova build ios --device --release
xcrun -v -sdk iphoneos PackageApplication "${WORKSPACE}/platforms/ios/build/device/My Cordova App.app" -o "${WORKSPACE}/platforms/ios/build/device/My Cordova App.ipa"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

###Visual Studio Specific Features
A quick way to get you project working with Visual Studio specific features outside of Cordova is to add the sample [Visual Studio Tools for Apache Cordova CLI Support Plugin](http://aka.ms/vstacoplugin) to your project.

To support the Task Explorer, the plugin takes advantage of the technique illustrated in the [Gulp tutorial](http://aka.ms/cordovagulp) for wiring in Gulp tasks to Cordova build events. To support res/native, the plugin uses a similar approach and we will briefly cover how this works behind the scenes.

##### Behind the Scenes: res/native Hook
If you are using Tools for Apache Cordova in Visual Studio, you may be taking advantage of the “res/native” folder in your project. This provides a useful way to update native project build artifacts without having to check in the platforms folder but is not currently a base Cordova CLI feature.

You can take a look at the sample sample [Tools for Apache Cordova CLI Support Plugin](http://aka.ms/vstacoplugin) for a quick way to add in support. The plugin takes advantage of something called a [Cordova hook](http://aka.ms/cordovahooks) that allows you to wire in build steps in the Cordova CLI build process. A Cordova hook can be implemented using either shell scripts (batch, bash, or node.js based) or as a node.js module.

For this behind the scenes tutorial, we’ll use a node.js shell script for simplicity sake. All that needs to happen is the following script should be placed in the “hooks/before\_prepare” folder. We will call the file “hook-res-native.js” but the filename itself does not matter.

Add the following to the file:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#!/usr/bin/env node
var fs = require('fs'),
	path = require('path');

process.env["CORDOVA_PLATFORMS"].split(",").forEach(function(platform) {
	console.log("Processing res/native for " + platform);
	var resNative = path.join(process.cwd(), "res", "native", platform)
	if (fs.existsSync(resNative)) {
		copyFiles(resNative, path.join(process.cwd(), "platforms", platform));
	}
});

// Recursive copy function for res/native processing
function copyFiles(srcPath, destPath) {
	if (fs.statSync(srcPath).isDirectory()) {
		if (!fs.existsSync(destPath)) {
			fs.mkdirSync(destPath);
		}
		fs.readdirSync(srcPath).forEach(function (child) {
			copyFiles(path.join(srcPath, child), path.join(destPath, child));
		});
	} else {
		fs.writeFileSync(destPath, fs.readFileSync(srcPath));
	}
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can place this into a "hooks\before_prepare" folder Visual Studio Cordova project and check it into source control and it will automatically be used.