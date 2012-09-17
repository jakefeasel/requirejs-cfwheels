#RequireJS for ColdFusion on Wheels

##About

[Coldfusion on Wheels](http://cfwheels.org) developers - 
	Do you want to use [RequireJS](http://requirejs.org) to organize and optimize your JavaScript code and CSS styles?  
	Do you want to use [{LESS}](http://lesscss.org) to build more efficient stylesheets?
	Have you been annoyed with the cumbersome build and optimization process when you want to deploy either of these to production?
	If yes, then this is the plugin for you!

The primary goal for this plugin is to allow simple, efficient transitions between your fully-expanded development environment and your optimized production environment.

##Basic JavaScript code setup

It is expected that you have done the essential RequireJS setup for your cfwheels/javascripts/ code.  Consult the [getting started guide for RequireJS](http://requirejs.org/docs/start.html#add) to see how to add your main script configuration file to your project, if you haven't done so already.  The important thing is that you **use an external script**, not inline code, for your calls to *require()*. 

Drop the file RequireJS-0.X.zip into your cfwheels/plugins folder and reload your app.  Also, if you are using URLRewriting="On" you will need to modify your web server config file to add other exceptions for "javascripts_min" and "stylesheets_min".  See the examples for .htaccess or web.config.  

Now you can start using the requirejsTag() helper function in your views:

	<cfoutput>#requirejsTag()#</cfoutput> 
		
This call will generate a script tag that will look one of two basic ways, depending on your environment mode.  

If in *design* or *development*, the script tag this will generate looks like this:

	<script type="text/javascript" src="plugins/RequireJS/require.js" data-main="javascripts/main"></script>
		
If in *testing* or *production*, script tag this will generate looks like this:

	<script type="text/javascript" src="plugins/RequireJS/require.js" data-main="javascripts_min/main"></script>
		
These are using the default options for the requirejsTag helper.  You can override the defaults easily enough, with these arguments:

	<cfoutput>
	#requirejsTag(
		src="path/to/your/copy/of/require.js",
		main="javascripts/myMain",
		build="path/to/your/build.js"
	)#
	</cfoutput>

I bundle a copy of require.js 2.0.4 (latest as of this writing) with the plugin, as well as a copy of js.jar from [Mozilla Rhino](https://github.com/mozilla/rhino).
		
If you want to use your own copy of require.js, specify it with the "src" argument.

I assume you want to use the require.js default config file "main.js" for your JS bundle (which would be located under "cfwheels/javascripts/main.js" based on the CF Wheels convention).  If you want to use a different config file to build your bundle, specify it with the "main" argument (omit the trailing .js).

If for some reason you want to use a different build file, I strongly suggest you do so by making a copy of my version (plugins/RequireJS/build.js) and just modify the "modules" section.  Changing the various directory settings will likely break things.

##Basic Stylesheets setup

For each set of stylesheets you'd like to embed on a page, you can embed them all together like so:

	<cfoutput>
	#requireStyleTags(
		href=["bootstrap.css","bootstrap-responsive.css","mainpage.less"], 
		outputTarget="built.css"
	)#
	</cfoutput>

When in *design* or *development*, this function will return link tags for each of your *href* array elements (these must be relative to the stylesheets folder, and include the file extension).  In this example, it would return this:

	<link href="stylesheets/bootstrap.css" media="all" rel="stylesheet" type="text/css" /><link href="stylesheets/bootstrap-responsive.css" media="all" rel="stylesheet" type="text/css" /><link href="stylesheets/mainpage.css" media="all" rel="stylesheet" type="text/css" />

When in *testing* or *production*, this function will return a single link tag, pointing to the outputTarget you specify (relative to the stylesheets_min folder).  In this example, it would return this:

	<link href="stylesheets_min/built.css" media="all" rel="stylesheet" type="text/css" />

This file will be a minified copy of each of the three original stylesheets, all concatonated into the single file for maximum efficiency.

Note - the outputTarget argument is optional.  If you omit it, it will build a target file based on the same name as the requested CGI.script_name.

##LESS Stylesheets

You may have noticed that in the above section, the last stylesheet in the href array was named "mainpage.**less**", but the link tag that was generated for it was named "mainpage.**css**".  This is the server-side LESS transformation in action, with no work on your part.  The plugin will check for the presence of a current (based on timestamps) .css file with the same name as the .less file you specify.  If we can't find a matching css file, or if the timestamp for the less file is newer than the css file, then the plugin will automatically execute the Java Rhino server-side JavaScript engine together with the less-rhino-1.3.0.js compiler in order to build a new version of the css file.  This means you can just write your less code and see the results in the browser immeidately upon your next refresh (assuming you're in design or development mode; otherwise you'll need to rebuild the optimized version - see below).


##Optimization

After you've done the above setup, you can trigger the optimization process by doing these things:

1. Put your environment in testing or production mode

2. Reload your app, and include *&optimizeRequireJS=true* as a URL parameter, like so:

    index.cfm?reload=true&password=foobar&optimizeRequireJS=true

This will start background threads on your server, running the optimization process for both your javascript and your stylesheets.  This process usually takes a bit of time to run (several minutes, so be sure your CF thread execution timeout is set to a decent amount; 10 minutes should be safe).

If you're watching closely after you invoke the optimization process, you'll notice that **initially, it returns the fully-expanded code**.  This is intentional. The idea is that, rather than potentially have your site broken while you wait for the new, optimized code to build, you will instead serve up the expanded code.  This won't be optimized, but at least it will work.  Then, when the optimization process finishes, all subsequent requests will start using the new bundle.

The build process creates a folder named javascripts_min under your app root.  This folder will contain a minified version of every .js file under your javascripts folder, including subfolders.  They will then be grouped together into a single file for your bundle (javascripts_min/main.js for example) which will be used from then on. A very similar folder named stylesheets_min will be created to hold your optimized CSS files, together with your single bundled target file.

If you have trouble with the build, check the file "cfwheels/plugins/RequireJS/optimize_output.txt" - this will contain errors or other build process output.  CSS errors would be viewable in the file "cfwheels/plugins/RequireJS/optimize_css_output.txt"

**Note** This process will not automatically build every bundle you might have throughout your app.  It only works on pages that use the requirejsTag() and/or the requireStyleTags() helpers, and then only those pages which were reloaded with the *&optimizeRequireJS=true* parameter passed.
This means if you have two different bundles, like so:

*views/foobar/index.cfm*

    #requirejsTag(main="javascripts/foobar_main")#
		
*views/blah/index.cfm*

    #requirejsTag(main="javascripts/blah_main")#
		
You'll have to invoke the optimization process for both of these actions individually, like so:

	index.cfm?controller=foobar&action=index&reload=true&password=myPass&optimizeRequireJS=true
	index.cfm?controller=blah&action=index&reload=true&password=myPass&optimizeRequireJS=true
		
It's important to mention that you'll have to wait until one finishes before you start the next one.  This is probably for the best, since the optimization process is fairly memory and processor intensive.

##Example Javascript Usage

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
 


##Example Stylesheet Usage

Not surprisingly, I'm also using this tag on SQLFiddle.com to manage my stylesheets.  Here is one example taken from there:

[views/layout.cfm](https://github.com/jakefeasel/sqlfiddle/blob/master/views/layout.cfm)

	<cfoutput>
		#requireStyleTags([
			"codemirror.css",
			"bootstrap.css",
			"bootstrap-responsive.min.css",
			"fiddle.css",
			"qp.css"	
		])#
	</cfoutput>

 
-- Jake Feasel, 09/16/2012
