

## Gruntfile.js (or .coffee)
The last required file (in addition to `.aerobatic`, `package.json`, and `index.html`) is your `Gruntfile.js`. There's plenty of documentation on the Grunt site, so this will only cover the Aerobatic specific setup.

```js
module.exports = function(grunt) {
  grunt.initConfig({
    watch: {
      options: {
        spawn: true,
        livereload: true
      },
      index: {
        files: ['index.html']
      },
      css: {
        files: ['css/*.css']
      },
      scripts: {
        files: ['js/**/*.js'],
        tasks: ['uglify']
      }
    }
  });


  grunt.registerTask('build', ['jshint', 'uglify', 'cssmin']);
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-watch');
  // Load other tasks
};
```  
