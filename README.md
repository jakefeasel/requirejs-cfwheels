#RequireJS for ColdFusion on Wheels

##About

[Coldfusion on Wheels](http://cfwheels.org) developers - 
	Do you want to use [RequireJS](http://requirejs.org) to organize and optimize your JavaScript code?  
	Have you been annoyed with the cumbersome optimization process when you want to deploy to production?
	If yes, then this is the plugin for you!

The primary goal for this plugin is to allow simple, efficient transitions between your fully-expanded development environment and your optimized production environment.

##Basic code setup

It is expected that you have done the essential RequireJS setup for your cfwheels/javascripts/ code.  Consult the [getting started guide for RequireJS](http://requirejs.org/docs/start.html#add) to see how to add your main script configuration file to your project, if you haven't done so already.  The important thing is that you **use an external script**, not inline code, for your calls to *require()*. 

Drop the file RequireJS-0.1.zip into your cfwheels/plugins folder and reload your app.  Also, if you are using URLRewriting="On" you will need to modify your web server config file to add another exception for "javascripts_min".  See the examples for .htaccess or web.config.  

Now you can start using the requirejsTag() helper function in your views:

	<cfoutput>#requirejsTag()#</cfoutput> 
		
This call will generate a script tag that will look one of two basic ways, depending on your environment mode.  

If in *design* or *development*, the script tag looks like this:

	<script type="text/javascript" src="plugins/RequireJS/require.js" data-main="javascripts/main"></script>
		
If in *testing* or *production*, script tag looks like this:

	<script type="text/javascript" src="plugins/RequireJS/require.js" data-main="javascripts_min/main"></script>
		
These are using the default options for the requirejsTag helper.  You can override the defaults easily enough, with these arguments:

	<cfoutput>
	#requirejsTag(
		src="path/to/your/copy/of/require.js"
		main="javascripts/myMain"
		build="path/to/your/build.js"
	)#
	</cfoutput>

I bundle a copy of require.js 2.0.4 (latest as of this writing) with the plugin, as well as a copy of js.jar from [Mozilla Rhino](https://github.com/mozilla/rhino).
		
If you want to use your own copy of require.js, specify it with the "src" argument.

I assume you want to use the require.js default config file "main.js" for your JS bundle (which would be located under "cfwheels/javascripts/main.js" based on the CF Wheels convention).  If you want to use a different config file to build your bundle, specify it with the "main" argument (omit the trailing .js).

If for some reason you want to use a different build file, I strongly suggest you do so by making a copy of my version (plugins/RequireJS/build.js) and just modify the "modules" section.  Changing the various directory settings will likely break things.

##Optimization

After you've done the above setup, you can trigger the optimization process by doing these things:

1. Put your environment in testing or production mode

2. Reload your app, and include *&optimizeRequireJS=true* as a URL parameter, like so:

    index.cfm?reload=true&password=foobar&optimizeRequireJS=true

This will start a background thread on your server, running the optimization process.  This process usually takes a bit of time to run (several minutes, so be sure your CF thread execution timeout is set to a decent amount; 10 minutes should be safe).

If you're watching closely after you invoke the optimization process, you'll notice that **initially, it returns the fully-expanded code**.  This is intentional. The idea is that, rather than potentially have your site broken while you wait for the new, optimized code to build, you will instead serve up the expanded code.  This won't be optimized, but at least it will work.  Then, when the optimization process finishes, all subsequent requests will start using the new bundle.

The build process creates a folder named javascripts_min under your app root.  This folder will contain a minified version of every .js file under your javascripts folder, including subfolders.  They will then be grouped together into a single file for your bundle (javascripts_min/main.js for example) which will be used from then on. 

If you have trouble with the build, check the file "cfwheels/plugins/RequireJS/optimize_output.txt" - this will contain errors or other build process output.

**Note** This process will not automatically build every bundle you might have throughout your app.  It only works on pages that use the requirejsTag() helper, and then only those pages which were reloaded with the *&optimizeRequireJS=true* parameter passed.
This means if you have two different bundles, like so:

*views/foobar/index.cfm*

    #requirejsTag(main="javascripts/foobar_main")#
		
*views/blah/index.cfm*

    #requirejsTag(main="javascripts/blah_main")#
		
You'll have to invoke the optimization process for both of these actions individually, like so:

	index.cfm?controller=foobar&action=index&reload=true&password=myPass&optimizeRequireJS=true
	index.cfm?controller=blah&action=index&reload=true&password=myPass&optimizeRequireJS=true
		
It's important to mention that you'll have to wait until one finishes before you start the next one.  This is probably for the best, since the optimization process is fairly memory and processor intensive.

##Example Usage

I built this plugin as part of my site, [SQL Fiddle](http://sqlfiddle.com).  You can take a look in [my github repository](https://github.com/jakefeasel/sqlfiddle) to see the whole picture, but the main thing you'll probably want to look at is my specific use of this plugin:

[javascripts/main.js](https://github.com/jakefeasel/sqlfiddle/blob/master/javascripts/main.js)

	requirejs.config({
	    shim: {
	        'backbone': {
				deps: ['underscore', 'jquery', 'json2'],
				exports: 'Backbone'
			},
			'mode/mysql/mysql': ['codemirror'],		
			'jquery.blockUI': ['jquery'],
			'jquery.cookie': ['jquery'],
			'bootstrap-collapse': ['jquery'],
			'bootstrap-tab': ['jquery'],
			'bootstrap-dropdown': ['jquery'],
			'bootstrap-modal': ['jquery'],
			'bootstrap-tooltip': ['jquery'],
			'bootstrap-popover': ['jquery','bootstrap-tooltip'],
			'oracle_xplan/loadswf': ['oracle_xplan/flashver'],
			
			/* Jake's code starts here: */
			'websql_driver': ['jquery', 'sqlite_driver'],
			'sqljs_driver': ['jquery', 'sqlite_driver'],
			'fiddle_backbone': ['backbone', 'mode/mysql/mysql', 'websql_driver', 'sqljs_driver', 'handlebars-1.0.0.beta.6', 'jquery.blockUI'],
			'dbTypes_cached': ['jquery','fiddle_backbone'],
			'ddl_builder': ['jquery','handlebars-1.0.0.beta.6','date.format'],
			'fiddle2': ['dbTypes_cached', 'ddl_builder', 'jquery.cookie', 'idselector', 'bootstrap-collapse', 'bootstrap-dropdown', 'bootstrap-modal', 'bootstrap-tooltip', 'bootstrap-popover']
		}
	});		
	
	
	require([	'jquery','underscore','json2','codemirror','bootstrap-tooltip','sqlite_driver',
				'backbone','mode/mysql/mysql','websql_driver','sqljs_driver','handlebars-1.0.0.beta.6','jquery.blockUI',
				'fiddle_backbone','date.format','dbTypes_cached','ddl_builder','jquery.cookie','idselector','oracle_xplan/flashver','oracle_xplan/loadswf', 
				'bootstrap-collapse','bootstrap-dropdown','bootstrap-modal','bootstrap-popover','bootstrap-tab','fiddle2'
			], function($) {
		
	});


[views/layout.cfm](https://github.com/jakefeasel/sqlfiddle/blob/master/views/layout.cfm)

	<!--- not much of an example here, I realize, but it does the job for me --->
	<cfoutput>#requirejsTag()#</cfoutput>
 

 Not much to it.
 
-- Jake Feasel, 07/16/2012