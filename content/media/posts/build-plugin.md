---
title: How to build webapp widget plugin
tags:
  - webapp
  - plugin
---
 
# The key points

- You can create the plugin uisng whatever framework you want.
- The important point is to bundle your package and prepare easy to use plugin entrypoint (UMD, ES).
- Remember about security. Almost every part of webapp is easy to access if you have enought knowledge.
- Serve your plugin static files throught CDN to speed up loading.

## The problem

Recentlt at the work I supervises development of the new app idea using AI. The time to first push to production come in, so i start preparing CI process. I know that it will be a bit different process than base webapp. To simplify, We want build the app in that way to be avaible to use in any page like other plugins:
```html
<!DOCTYPE html>
<html lang="en">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>My page</title>
</head>

<body>
	<script src="https://ai-plugin/plugin.umd.js"></script>
	<script>
		// Initialize the plugin with configuration
		loadPlugin({
			apiKey: 'my-api-key',
			config:{
				color:...
			}
		})
	</script>
</body>

</html>
```

