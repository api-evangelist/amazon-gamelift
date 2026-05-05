---
title: "Unreal Engine 5 dedicated server development with Amazon GameLift Anywhere"
url: "https://aws.amazon.com/blogs/gametech/unreal-engine-5-dedicated-server-development-with-amazon-gamelift-anywhere/"
date: "Thu, 04 May 2023 18:16:03 +0000"
author: "AWS for Games Content Team"
feed_url: "https://aws.amazon.com/blogs/gametech/tag/amazon-gamelift/feed/"
---
<p>Building dedicated servers is challenging, and developers need the ability to quickly test and iterate on their games. The recently updated <a href="https://aws.amazon.com/gamelift/">Amazon GameLift</a> Server SDK from Amazon Web Services (AWS) provides an <a href="https://www.unrealengine.com/">Unreal Engine</a> plugin that is compatible with both Unreal Engine 4 and Unreal Engine 5.</p> 
<p>The plugin makes it easy for Unreal Engine developers to use an <a href="https://docs.aws.amazon.com/gamelift/latest/developerguide/fleets-creating-anywhere.html">Amazon GameLift Anywhere</a> fleet to test their game server on a local workstation without the need to upload a build to Amazon GameLift. Amazon GameLift Anywhere fleets also integrate with <a href="https://docs.aws.amazon.com/gamelift/latest/flexmatchguide/match-intro.html">Amazon GameLift FlexMatch</a> and <a href="https://docs.aws.amazon.com/gamelift/latest/developerguide/queues-creating.html">Queues</a>, allowing end-to-end testing.</p> 
<p>This post describes how to integrate the <a href="https://aws.amazon.com/gamelift/getting-started/">Amazon GameLift Server SDK 5.0</a> plugin into Unreal Engine 5, create an Anywhere fleet, and register your local workstation as a compute resource. You will then create a game session running on your local workstation, saving time and shortening the game development feedback loop.</p> 
<h3>Solution overview</h3> 
<p>Unreal Engine 5 offers a free sample game project called Lyra that provides online multiplayer support. This project is a great starting point for building a dedicated server, and in this walkthrough, we integrate the Amazon GameLift plugin into Lyra and describe how to use Amazon GameLift Anywhere to test and iterate on your game server.</p> 
<p><img alt="Unreal Engine 5 Lyra Starter Game" class="aligncenter size-full wp-image-4943" height="526" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-01.png" width="936" /></p> 
<p><em><a href="https://www.unrealengine.com/marketplace/en-US/product/lyra">Unreal Engine 5 Lyra Starter Game</a></em></p> 
<h3>Walkthrough</h3> 
<p>This walkthrough takes you through the following steps:</p> 
<ul> 
 <li>Build Unreal Engine from source</li> 
 <li>Build the Lyra Starter Game</li> 
 <li>Set up the Amazon GameLift plugin</li> 
 <li>Configure the Lyra Starter Game to use the plugin</li> 
 <li>Set up an Amazon GameLift Anywhere fleet</li> 
 <li>Create a game session</li> 
</ul> 
<h3>Prerequisites</h3> 
<p>For this walkthrough, you should have the following:</p> 
<ul> 
 <li>&nbsp;An <a href="https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&amp;client_id=signup">AWS account</a></li> 
 <li>A GitHub account</li> 
 <li>An Epic Games account</li> 
 <li>Visual Studio 2022</li> 
 <li><a href="https://wiki.openssl.org/index.php/Binaries">OpenSSL</a> and <a href="https://cmake.org/download/">cmake</a> installed</li> 
 <li>Prior experience with Unreal Engine and C++</li> 
</ul> 
<h4><em>Build Unreal Engine from source</em></h4> 
<p>In order to build a dedicated server, you need to compile Unreal Engine from source. This is required so that you can specify a Server Target for the build. Unreal Engine 5.1.0 is used throughout this blog post.</p> 
<p><em>To build your desired version of Unreal Engine</em></p> 
<ol> 
 <li>Follow these steps to link your GitHub account to your Epic Games account.</li> 
 <li>Clone your desired version of the Unreal GitHub repo by running:</li> 
</ol> 
<p><code>git clone -b 5.1.0-release --single-branch https://github.com/EpicGames/UnrealEngine.git</code></p> 
<ol start="3"> 
 <li>If you plan to cross-compile for Linux, you will need to install the <a href="https://docs.unrealengine.com/5.1/en-US/linux-development-requirements-for-unreal-engine/#cross-compiletoolchain">cross-compile toolchain</a>.</li> 
 <li>Follow the Unreal Engine documentation to <a href="https://docs.unrealengine.com/5.1/en-US/building-unreal-engine-from-source/">build the engine from source</a>.</li> 
</ol> 
<h3>Build the Lyra Starter Game</h3> 
<h4><em>To download Lyra</em></h4> 
<ol> 
 <li>Open the Epic Games Launcher</li> 
 <li>Click on the Marketplace tab and search for “Lyra”</li> 
 <li>Click “Create Project”</li> 
 <li>Select your Unreal Engine version and click “Create”</li> 
</ol> 
<h4><em>To build Lyra</em></h4> 
<ol> 
 <li>Right-click on the <code>LyraStarterGame</code> project file and select “Switch Unreal Engine version…”. Select your Unreal source build.</li> 
</ol> 
<p><img alt="Image showing right-click menu and “Generate Visual Studio project files” highlighted" class="aligncenter size-full wp-image-4944" height="196" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-02.png" width="882" /></p> 
<ol> 
 <li>Right-click on the <code>LyraStarterGame</code> project file and select “Generate Visual Studio project files”.</li> 
 <li>Open Lyra project in the Unreal Editor.</li> 
 <li>Follow the <a href="https://docs.unrealengine.com/5.1/en-US/setting-up-dedicated-servers-in-unreal-engine/">dedicated server tutorial</a> in the Unreal documentation to configure Lyra and build, package, and test both a server and a client.</li> 
</ol> 
<h3>Set up the Amazon GameLift plugin</h3> 
<p>The Amazon GameLift plugin supports building both Windows and Linux servers. In both cases, the Managed Servers C++ SDK must be compiled, and the library files copied into the plugin folder.</p> 
<h4><em>Download the Amazon Gamelift plugin</em></h4> 
<ol> 
 <li>Download the latest version of the <a href="https://aws.amazon.com/gamelift/getting-started/">Amazon GameLift Managed Servers C++ SDK</a>.</li> 
 <li>Unzip the archive.</li> 
</ol> 
<h4><em>To prepare the plugin for Windows servers</em></h4> 
<ol> 
 <li>Open the Visual Studio 2022 <a href="https://learn.microsoft.com/en-us/visualstudio/ide/reference/command-prompt-powershell?view=vs-2022">Developer Command Prompt</a>.</li> 
 <li>Change directory to the C++ SDK folder.</li> 
 <li>Build the Windows library by running:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">mkdir out
cd out 
cmake -G "Visual Studio 17 2022" -DBUILD_FOR_UNREAL=1 .. 
msbuild ALL_BUILD.vcxproj /p:Configuration=Release</code></pre> 
</div> 
<ol start="4"> 
 <li>This creates 2 library files in the out\gamelift-server-sdk\Release folder:</li> 
</ol> 
<p><code>aws-cpp-sdk-gamelift-server.lib</code></p> 
<p><code>aws-cpp-sdk-gamelift-server.dll</code></p> 
<ol start="5"> 
 <li>Copy these files into the Amazon GameLift Unreal plugin package in <code>ThirdParty\GameLiftServerSDK\Win64</code></li> 
</ol> 
<h4><em>Prepare the plugin for Linux servers</em></h4> 
<p>Optionally, you can build and integrate the Linux version of the library, so that the plugin also supports Linux servers.</p> 
<ol> 
 <li>Change directory to the C++ SDK folder.</li> 
 <li>Use a machine or a docker container running Amazon Linux 2 to build the Linux version of the library by running:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">mkdir out
cd out 
cmake -DBUILD_FOR_UNREAL=1 .. 
make</code></pre> 
</div> 
<ol start="3"> 
 <li>This creates a library file <code>libaws-cpp-sdk-gamelift-server.so in the out/gamelift-server-sdk</code> folder</li> 
 <li>Copy this file into the Amazon GameLift Unreal plugin package in: <code>ThirdParty\GameLiftServerSDK\Linux\x86_64-unknown-linux-gnu</code></li> 
</ol> 
<h3>Configure the Lyra Starter Game to use the plugin</h3> 
<p>With the plugin ready to use:</p> 
<ul> 
 <li>Enable the plugin in the Lyra Starter Game project</li> 
 <li>Add code to use the Amazon GameLift SDK</li> 
</ul> 
<h4><em>Enabling the plugin</em></h4> 
<ol> 
 <li>Copy the top-level <code>GameLiftServerSDK</code> folder into the <code>LyraStarterGame\Plugins</code> folder. The folder structure should look like the following:</li> 
</ol> 
<p><img alt="Image showing Unreal folder tree with Plugins\GameLiftServerSDK folder highlighted" class="aligncenter size-full wp-image-4945" height="562" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-03.png" width="770" /></p> 
<ol start="2"> 
 <li>Enable the plugin by editing the LyraStarterGame.uproject file and adding the GameLiftServerSDK plugin to the bottom of the list of plugins:</li> 
</ol> 
<p><img alt="Image showing GameLiftServerSDK plugin node in the LyraStarterGame.uproject file" class="aligncenter size-full wp-image-4946" height="232" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-04.png" width="478" /></p> 
<ol start="3"> 
 <li>Edit the Source<code>\LyraGame\LyraGame.Build.cs</code> file and add “<code>GameLiftServerSDK</code>” to the list of public module dependencies:</li> 
</ol> 
<p><img alt="Image showing GameLift ServerSDK added to public module dependencies list" class="aligncenter size-full wp-image-4947" height="670" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-05.png" width="642" /></p> 
<ol start="4"> 
 <li>Open the LyraStarterGame project in Unreal Editor and allow it to rebuild the plugin on startup.</li> 
</ol> 
<h4><em>Using the Amazon GameLift Server SDK</em></h4> 
<p>When you call <code>InitSDK()</code> to initialize the Amazon GameLift Server SDK, you need to provide a <code>FServerParameters</code> object, containing values for how the SDK should be initialized.</p> 
<p>With managed <a href="https://aws.amazon.com/ec2/">Amazon EC2</a> instances, these server parameter values can be left empty.</p> 
<p>However, when setting up an Amazon GameLift Anywhere fleet, you need to specify server parameters to send to Amazon GameLift:</p> 
<ul> 
 <li>The Host ID/Compute Name of the Amazon GameLift Anywhere instance</li> 
 <li>The Anywhere Fleet ID</li> 
 <li>The WebSocket URL</li> 
 <li>The Process ID of the running process</li> 
 <li>The compute authorization token</li> 
</ul> 
<p>When you want to test your server as part of an Amazon GameLift Anywhere fleet, you can provide these parameters via the command line.&nbsp; When you deploy to a managed Amazon EC2 fleet, you can simply omit these parameters.</p> 
<p>You will add code to:</p> 
<ul> 
 <li>Initialize the Amazon GameLift Server SDK via the InitSDK action, using our command line parameters, if provided.</li> 
 <li>Inform the Amazon GameLift service that our process is ready to host players via the ProcessReady action, implementing callback functions to handle requests from the Amazon GameLift service to check the process health, activate a new game session, and terminate a running game session.</li> 
</ul> 
<ol> 
 <li>Open Source\LyraGame\LyraGameMode.h and add a private function definition for InitGameLift(). Add a definition to override the BeginPlay() protected function:<br /> <img alt="Image showing function definitions added to LyraGameMode.h" class="aligncenter size-full wp-image-4948" height="347" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-06.png" width="936" /></li> 
</ol> 
<ol start="2"> 
 <li>Open <code>Source\LyraGame\LyraGameMode.cpp</code> and include the Amazon GameLift Server SDK header file at the top of the file:</li> 
</ol> 
<p><code>#include “GameLiftServerSDK.h”</code></p> 
<p><img alt="Image showing GameLiftServerSDK.h included at the top of the file" class="aligncenter wp-image-4949" height="120" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-07.jpg" width="510" /></p> 
<ol start="3"> 
 <li>Add the code for the <code>BeginPlay</code> and <code>InitGameLift</code> functions at the bottom of the file:</li> 
</ol> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-cpp">void ALyraGameMode::BeginPlay()
{
	// Code enclosed with the WITH_GAMELIFT=1 flag is only processed if the following are true:
	// The plugin found the Amazon GameLift server SDK binary files.
	// The build is a game server : Target.Type == TargetRules.TargetType.Server

#if WITH_GAMELIFT
	InitGameLift();
#endif
}

void ALyraGameMode::InitGameLift()
{
	FString AuthToken;
	FString HostId;
	FString FleetId;
	FString WSSUrl;

	FServerParameters serverParameters;

	UE_LOG(LogLyra, Log, TEXT("Running on port %d"), GetWorld()-&gt;URL.Port);

	FParse::Value(FCommandLine::Get(), TEXT("-authtoken="), serverParameters.m_authToken);

	if (FParse::Value(FCommandLine::Get(), TEXT("-hostid="), serverParameters.m_hostId))
	{
		UE_LOG(LogLyra, Log, TEXT("HOST_ID: %s"), *serverParameters.m_hostId)
	}

	if (FParse::Value(FCommandLine::Get(), TEXT("-fleetid="), serverParameters.m_fleetId))
	{
		UE_LOG(LogLyra, Log, TEXT("FLEET_ID: %s"), *serverParameters.m_fleetId)
	}

	if (FParse::Value(FCommandLine::Get(), TEXT("-websocketurl="), serverParameters.m_webSocketUrl))
	{
		UE_LOG(LogLyra, Log, TEXT("WEBSOCKET_URL: %s"), *serverParameters.m_webSocketUrl)
	}

	serverParameters.m_processId = FString::Printf(TEXT("%d"), GetCurrentProcessId());
	UE_LOG(LogLyra, Log, TEXT("PID: %s"), *serverParameters.m_processId);

	UE_LOG(LogLyra, Log, TEXT("Initializing the GameLift Server"));

	//Getting the module first.
	FGameLiftServerSDKModule* gameLiftSdkModule = &amp;FModuleManager::
		LoadModuleChecked&lt;FGameLiftServerSDKModule&gt;(FName("GameLiftServerSDK"));

	//InitSDK will establish a local connection with GameLift's agent to enable further communication.
	FGameLiftGenericOutcome initSdkOutcome = gameLiftSdkModule-&gt;InitSDK(serverParameters);
	if (initSdkOutcome.IsSuccess())
	{
		UE_LOG(LogLyra, Log, TEXT("GameLift InitSDK succeeded"));
	}
	else
	{
		UE_LOG(LogLyra, Log, TEXT("ERROR: InitSDK failed"));
		FGameLiftError gameLiftError = initSdkOutcome.GetError();
		UE_LOG(LogLyra, Log, TEXT("ERROR: %s"), *gameLiftError.m_errorMessage);
		return;
	}

	//When a game session is created, GameLift sends an activation request to the game server and passes along the game session object containing game properties and other settings.
	//Here is where a game server should take action based on the game session object.
	//Once the game server is ready to receive incoming player connections, it should invoke GameLiftServerAPI.ActivateGameSession()
	auto onGameSession = [=](Aws::GameLift::Server::Model::GameSession gameSession)
	{
		FString gameSessionId = FString(gameSession.GetGameSessionId());
		UE_LOG(LogLyra, Log, TEXT("GameSession Initializing: %s"), *gameSessionId);
		gameLiftSdkModule-&gt;ActivateGameSession();
	};
	FProcessParameters* params = new FProcessParameters();
	params-&gt;OnStartGameSession.BindLambda(onGameSession);

	//OnProcessTerminate callback. GameLift will invoke this callback before shutting down an instance hosting this game server.
	//It gives this game server a chance to save its state, communicate with services, etc., before being shut down.
	//In this case, we simply tell GameLift we are indeed going to shutdown.
	params-&gt;OnTerminate.BindLambda([=]() {
		UE_LOG(LogLyra, Log, TEXT("Game Server Process is terminating"));
		gameLiftSdkModule-&gt;ProcessEnding();
		});

	//This is the HealthCheck callback.
	//GameLift will invoke this callback every 60 seconds or so.
	//Here, a game server might want to check the health of dependencies and such.
	//Simply return true if healthy, false otherwise.
	//The game server has 60 seconds to respond with its health status. GameLift will default to 'false' if the game server doesn't respond in time.
	//In this case, we're always healthy!
	params-&gt;OnHealthCheck.BindLambda([]() {
		UE_LOG(LogLyra, Log, TEXT("Performing Health Check"));
		return true;
		});

	//The game server takes the port from the Unreal world.
	params-&gt;port = GetWorld()-&gt;URL.Port;

	//Here, the game server tells GameLift what set of files to upload when the game session ends.
	//GameLift will upload everything specified here for the developers to fetch later.

	TArray&lt;FString&gt; logfiles;
	logfiles.Add(TEXT("LyraStarterGame/Saved/Logs/LyraStarterGame"));
	params-&gt;logParameters = logfiles;
	//Calling ProcessReady tells GameLift this game server is ready to receive incoming game sessions!
	UE_LOG(LogLyra, Log, TEXT("Calling Process Ready"));
	FGameLiftGenericOutcome processReadyOutcome = gameLiftSdkModule-&gt;ProcessReady(*params);
	if (processReadyOutcome.IsSuccess())
	{
		UE_LOG(LogLyra, Log, TEXT("Process Ready Succeded"));
	}
	else
	{
		UE_LOG(LogLyra, Log, TEXT("ERROR: Process Ready Failed"));
		FGameLiftError processReadyError = processReadyOutcome.GetError();
		UE_LOG(LogLyra, Log, TEXT("ERROR: %s"), *processReadyError.m_errorMessage);
	}
	UE_LOG(LogLyra, Log, TEXT("Init GameLift complete"));
}
</code></pre> 
  <pre></pre> 
 </div> 
</div> 
<ol start="4"> 
 <li>Build with a <code>Development Server</code> Solution Configuration in Visual Studio 2022.</li> 
</ol> 
<p><img alt="Image showing Development Server solution configuration highlighted" class="aligncenter size-full wp-image-4950" height="702" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-08.png" width="324" /></p> 
<ol start="5"> 
 <li>Package the server in the Unreal Editor.</li> 
</ol> 
<h3>Set up an Amazon GameLift Anywhere fleet</h3> 
<p>Now that you have your dedicated server ready, the next step is to create an Amazon GameLift Anywhere fleet that allows you to place game sessions on &nbsp;your local workstation.</p> 
<h4><em>Create an Amazon GameLift Anywhere fleet</em></h4> 
<ol> 
 <li>Create a custom location and specify a location name. Custom location names must begin with “custom-“ and must be a minimum of 8 characters in length:</li> 
</ol> 
<p><code>aws gamelift create-location --location-name custom-&lt;locationName&gt;</code></p> 
<ol start="2"> 
 <li>Create a fleet using the newly created location, specifying the compute type as ANYWHERE:</li> 
</ol> 
<p><code>aws gamelift create-fleet --name &lt;FleetName&gt; --compute-type ANYWHERE --locations "Location=&lt;LocationName&gt;"</code></p> 
<ol start="3"> 
 <li>You will be returned some attributes of the fleet – you will need the <code>FleetId</code> for the next step.</li> 
</ol> 
<h4><em>Register your workstation and start the server</em></h4> 
<ol> 
 <li>Register your local workstation as a compute resource in the fleet:</li> 
</ol> 
<p><code>aws gamelift register-compute --compute-name &lt;ComputeName&gt; --fleet-id &lt;FleetId&gt; --ip-address &lt;workstationIpAddress&gt; --location &lt;LocationName&gt;</code></p> 
<p>You will be returned a JSON object with the compute attributes. Make a note of the <code>GameLiftServiceSdkEndpoint</code> – you will need to pass this in to your game server.</p> 
<ol start="2"> 
 <li>Obtain a compute authorization token for your local workstation:</li> 
</ol> 
<p><code>aws gamelift get-compute-auth-token --fleet-id &lt;FleetId&gt; --compute-name &lt;ComputeName&gt;</code></p> 
<p>You will be returned a JSON object with an AuthToken which is required to authorize the server with Amazon GameLift when you initialize the Amazon GameLift Server SDK.</p> 
<p><strong>NOTE</strong> – the <code>AuthToken</code> expires 15 minutes after creation. Once the token expires, you will need to retrieve a new <code>AuthToken</code> for a different process.</p> 
<ol start="3"> 
 <li>Start the Lyra game server, passing in the compute name, the <code>GameLiftServiceSdkEndpoint</code>, the <code>AuthToken</code> and the <code>FleetId</code>:</li> 
</ol> 
<p><code>LyraServer.exe -log -fleetid=&lt;FleetId&gt; -hostid=&lt;ComputeName&gt; -websocketurl=&lt;GameLiftServiceSdkEndpoint&gt; -authtoken=&lt;AuthToken&gt;</code></p> 
<ol start="4"> 
 <li>You should see logs indicating that the call to <code>InitSDK</code> has succeeded and <code>ProcessReady</code> has been successfully called:</li> 
</ol> 
<p><img alt="Image showing logs to indicating that the InitSDK call was successful" class="aligncenter size-full wp-image-4951" height="299" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-09.png" width="936" /></p> 
<p>If the call to <code>InitSDK</code> fails, check the command line parameters and make sure you have provided the correct details, and the <code>AuthToken</code> has not expired.</p> 
<h3>Create a game session</h3> 
<p>The final step is to create a game session running on your fleet.</p> 
<p>In production, it is <em><strong>always</strong> </em>recommended to use <a href="https://docs.aws.amazon.com/gamelift/latest/developerguide/queues-intro.html">game session queues</a> and use <a href="https://docs.aws.amazon.com/gamelift/latest/developerguide/queue-events.html">game session placement events</a> to track the progress of your queue placement.</p> 
<p>In development, you can do this via the create-game-session console command:</p> 
<p><code>aws gamelift create-game-session --fleet-id &lt;FleetId&gt; --maximum-player-session-count 2 --location &lt;LocationName&gt;</code></p> 
<p>You will be returned a JSON object with a Status that will be ACTIVATING if everything worked correctly.</p> 
<p>You should also see an entry appear in your game server logs showing that the game server is activating:</p> 
<p><img alt="Image showing log of game session activating." class="aligncenter size-full wp-image-4952" height="37" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/05/04/uegl-10.jpg" width="1430" /></p> 
<p>You can now run the Lyra Starter game client to connect in to your server:</p> 
<p><code>LyraClient.exe -log</code></p> 
<h3>Conclusion</h3> 
<p>You have now learned to integrate the Amazon GameLift Server SDK into Unreal Engine 5 to start game sessions and carry out local testing via Amazon GameLift Anywhere.</p> 
<p>To learn more, please visit these additional resources:</p> 
<p><a href="https://aws.amazon.com/marketplace/pp/prodview-iu7szlwdt5clg?sr=0-1&amp;ref_=beagle&amp;applicationId=AWSMPContessa">AWS Marketplace | Unreal Engine 5</a></p> 
<p><a href="https://aws.amazon.com/about-aws/whats-new/2023/04/amazon-gamelift-unreal-engine-5/">Amazon GameLift adds support for Unreal Engine 5</a></p> 
<p><a href="https://aws.amazon.com/blogs/aws/introducing-amazon-gamelift-anywhere-run-your-game-servers-on-your-own-infrastructure/">Introducing Amazon GameLift Anywhere</a></p> 
<p><a href="https://docs.aws.amazon.com/gamelift/latest/developerguide/fleets-creating-anywhere.html">Amazon GameLift Anywhere Documentation</a></p> 
<p><a href="https://www.youtube.com/watch?v=lzEnqemZV74">Deploying an Anywhere Fleet on Amazon GameLift Tutorial Video</a></p> 
<p><a href="https://docs.unrealengine.com/5.1/en-US/setting-up-dedicated-servers-in-unreal-engine/">Setting Up Dedicated Servers | Unreal Documentation</a></p> 
<p><a href="https://docs.unrealengine.com/5.1/en-US/lyra-sample-game-in-unreal-engine/">Unreal Engine 5 Lyra Starter Game | Unreal Documentation</a></p> 
<p><a href="https://docs.unrealengine.com/5.1/en-US/multiplayer-programming-quick-start-for-unreal-engine/">Multiplayer Programming | Unreal Documentation</a></p>
