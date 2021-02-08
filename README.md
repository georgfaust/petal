# PETAL Stack Setup HOWTO

A log of commands used to setup Alpine.js and tailwindcss for Phoenix LiveView.


## check phoenix and node version

this combination should work.

```
$ mix phx.new --version
Phoenix v1.5.7

$ node --version
v14.15.4
```

NOTE: I was able generate a project with node 15.8, but ran into a bunch of errors later, so I downgraded to node 14.


## create a new live-view project

I omit ecto and dashboard to keep things simple

```
$ mix phx.new petal --live --no-ecto --no-dashboard
```

check if everything works with

```
petal/$ mix phx.server
```

first commit.

```
petal/$ git init
petal/$ git add * 
petal/$ git commit -a -m generator
```

## cleanup

### don't use sass and remove phoenix styles

- rename `app.scss` to `app.css` (also adjust link in `app.js`)
- delete `phoenix.css` (and its import in `app.css`)

### remove content from generated leex-files

- remove `<header>` from `root.hmtl.leex`
- replace `page_live.hmtl.leex` with `<div>Phoenix, Elixir, ________, _________ and LiveView!</div>`


```
petal/$ git commit -a -m cleanup
```

## format

just formatting some files for clean diffs.

```
petal$ git commit -a -m format
```

## install Alpine.js

see: https://dockyard.com/blog/2020/12/21/optimizing-user-experience-with-liveview

```
petal/assets$ npm install alpinejs
```

add some magic to app.js

```
petal/assets$ git diff HEAD js/app.js
 ...
 import "../css/app.css"
+import Alpine from "alpinejs";
 ...
-let liveSocket = new LiveSocket("/live", Socket, { params: { _csrf_token: csrfToken } })
+let liveSocket = new LiveSocket("/live", Socket, {
+   params: { _csrf_token: csrfToken },
+   dom: {
+   	onBeforeElUpdated(from, to) {
+       	if (from.__x) {
+           	Alpine.clone(from.__x, to);
+            }
+       },
+   },
+}
+)
```

change `page_live.html.leex` to

```
<div x-data="{}">
	<div>Phoenix, Elixir, ________, <span x-text="'Alpine.js'"></span> and Liveview</div>
</div>
```

to see that Alpine.js is working.

```
petal$ git commit -a -m alpinejs
```


## install tailwind


I'm installing tailwind as a postcss-plugin also using postcss as preprocessor

- https://tailwindcss.com/docs/installation#installing-tailwind-css-as-a-post-css-plugin
- https://tailwindcss.com/docs/using-with-preprocessors#using-post-css-as-your-preprocessor

A very thorough guide by Mike Clark can be found here:
https://pragmaticstudio.com/tutorials/adding-tailwind-css-to-phoenix

```
petal/assets$ npm install tailwindcss postcss postcss-import autoprefixer postcss-loader@4.2.0 --save-dev
```

NOTE: can't install postcss-loader@latest (v5.0) with a phoenix 1.5.7 generated project, because it requires webpack v5. 

### add tailwind to app.css

```
petal/assets$ git diff HEAD css/app.css
...
+@import "tailwindcss/base";
+@import "tailwindcss/components";
+@import "tailwindcss/utilities";
+
 @import "../node_modules/nprogress/nprogress.css";
```

### setup CSS-purge

```
petal/assets$ npx tailwindcss init
# -> creates `tailwind.config.js`
```

add paths to all files that may use tailwind classes to the `purge` array.

```
petal/assets$ cat tailwind.config.js 
module.exports = {
	purge: [
		'../lib/**/*.ex',
		'../lib/**/*.leex',
		'../lib/**/*.eex',
		'./js/**/*.js'
	],
...

petal/assets$ git diff HEAD package.json
...
    "scripts": {
-       "deploy": "webpack --mode production",
+       "deploy": "NODE_ENV=production webpack --mode production",
```


### configure postcss

```
petal/assets$ cat postcss.config.js 
module.exports = {
        plugins: [
                require('postcss-import'),
                require('tailwindcss'),
                require('autoprefixer'),
        ]
}

petal/assets$ git diff HEAD webpack.config.js
...
module.exports = (env, options) => {
	use: [
			MiniCssExtractPlugin.loader,
			'css-loader',
+			'postcss-loader',
			'sass-loader',
	],
}
```

### use tailwind

change `page_live.html.leex` to

```
<div x-data="{}">
	<div>Phoenix, Elixir, 
	    <span class="animate-ping">Tailwind</span>, 
	    <span x-text="'Alpine.js'"></span> 
	    and LiveView
	</div>
</div>
```

to see that tailwind is working.

```
petal$ git commit -a -m tailwind
```
