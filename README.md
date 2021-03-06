# Starter-kit for Assemble with Gulp

generating webpages with Assemble-beta and Gulp

Assemble with Gulp is at the moment of writing not quite ready for production. When you go to [the repo for gulp-assemble](https://github.com/assemble/gulp-assemble) you will be welcomed by a disclaimer. Things like rendering of markdown to html doesn't quite work yet, so better stay with the stable Grunt version for now. However on a small project with a Gulp workflow and markdown rendering not being required, you can give it a try.

The examples here are with using the beta version of Assemble **not** the plugin – _gulp-assemble_.

After `cd my-project` and `npm init` to generate your `package.json` you can follow the instructions from [the Readme of the repo](https://github.com/assemble/assemble#README.md).

**Install globally with [npm](npmjs.org)**

```bash
npm i -g assemble@beta
```

**Install locally with [npm](npmjs.org)**

```bash
npm i assemble@beta --save
```

Now you want to at least also install the `gulp-extname` plugin, to be able to dynamically rewrite file-extensions, otherwise your `index.hbs` will _not_ get the `.html` extension after doing: `assemble`

```bash
npm i gulp-extname --save-dev
```

The current workflow for this beta version is to have a seperate file for Assemble called `assemblefile.js`.

### Example assemblefile.js

```js
var assemble = require('assemble');
var extname = require('gulp-extname');

assemble.option('layout', 'base');

assemble.partials('src/templates/partials/*.hbs');
assemble.layouts('src/templates/layouts/*.hbs');
assemble.data(['src/data/*.{json,yml}']);

assemble.task('default', function () {
  assemble.src('src/content/**/*.hbs', { layout: 'base' })
    .pipe(extname())
    .pipe(assemble.dest('dist/static/'));
});
```

Go to the files in the repo to see about the new syntax in the `src/template` folder.
Here Assemble generates the `index.html` file unminified in the `dist/static/` folder after executing the `assemble` command on the command-line. Now we can use gulp-plugins to render and watch the `Sass` into `CSS`, have vendor prefixes automatically attached, and minify the `index.html` generated earlier by Assemble. Here I separated them out in tasks, with an added task for deployment to a gh-pages branch (only on the remote), which is generated by the `gulp-gh-pages` plugin (_so good_), one can see [the result here](http://atelierbram.github.io/Starter-Assemble-Gulp/).

```bash
npm install --save-dev gulp-gh-pages
// Usage: Define a deploy task in your gulpfile.js (as below) which can be used to push to gh-pages going forward.
```

### Example Gulpfile.js
```js
var gulp = require('gulp');
var sass = require('gulp-sass');
var sourcemaps = require('gulp-sourcemaps');
var autoprefixer = require('gulp-autoprefixer');
var htmlmin = require('gulp-htmlmin');

var ghPages = require('gulp-gh-pages');

// Deploy to ghPages on GitHub with `gulp deploy`
gulp.task('deploy', function() {
  return gulp.src('dist/**/*')
    .pipe(ghPages());
});

// option Vars
var srcAssets = 'src/assets/';
var distAssets = 'dist/assets/';
var sassOptions = {
  errLogToConsole: true,
  outputStyle: 'compressed'
};

// https://github.com/ai/browserslist#queries
var autoprefixerOptions = {
  browsers: ['last 2 versions', '> 1%', 'Firefox ESR', 'ie >= 9']
};

// minify html
gulp.task('minify', function() {
  return gulp.src('dist/static/*.html')
    .pipe(htmlmin({collapseWhitespace: true}))
    .pipe(gulp.dest('dist'))
});

// a DIY copy task, no need for a plugin with Gulp!
gulp.task('copy-js', function() {
  gulp.src('src/assets/js/*.js')
  .pipe(gulp.dest('dist/assets/js/'));
});

// tasks
gulp.task('sass', function () {
  return gulp
    // Find all `.scss` file from the `assets/sass` folder
    .src(srcSass)
    // Run Sass on those files
    .pipe(sass(sassOptions).on('error', sass.logError))
    .pipe(sourcemaps.write('./maps'))
    .pipe(autoprefixer(autoprefixerOptions))
    .pipe(autoprefixer())
    // write the resulting CSS in the dist-output folder for distribution
    .pipe(gulp.dest(distCss));
});

gulp.task('watch', function() {
  return gulp
    // Watch the Sass src folder for change,
    // and run `sass` task when something happens
    .watch(srcSass, ['sass'])
    // When there is a change,
    // log a message in the console
    .on('change', function(event) {
      console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
    });
});


gulp.task('default', ['sass', 'watch']);

```

In my `.bashrc` I have aliases defined for the commands which are defined above:

```bash
alias gls="gulp sass"
alias glw="gulp watch"
alias gla="assemble"
alias glm="gulp minify"
alias gld="gulp deploy"
alias glcpjs="gulp copy-js"
```

The assemble – and minify-task should ideally be combined into one task, but this has got to do for now.

#### Resources
- [Assemble 6.0 dev](https://github.com/assemble/assemble/tree/v0.6.0-dev)
- [gulp-extname](https://github.com/jonschlinkert/gulp-extname)
- [example Gulpfile.js](https://github.com/assemble/gulp-assemble/blob/master/gulpfile.js)
- [example assemblefile.js](https://gist.github.com/jonschlinkert/e2da295ec7ca5d159914)
- [gulp for beginners](https://css-tricks.com/gulp-for-beginners)
- [A Simple Gulp’y Workflow For Sass](http://www.sitepoint.com/simple-gulpy-workflow-sass/)
- [Getting started with Gulp and Sass](http://ryanchristiani.com/getting-started-with-gulp-and-sass/)
- [Introduction to Gulp.js 4: Creating CSS with Sass](http://stefanimhoff.de/2014/gulp-tutorial-4-css-generation-sass/)
- [Build Podcast](http://build-podcast.com/gulp/)
- [Advanced Gulp File](http://www.mikestreety.co.uk/blog/an-advanced-gulpjs-file)
- [Gulp docs on Github](https://github.com/gulpjs/gulp/blob/master/docs/README.md#articles-and-recipes)
