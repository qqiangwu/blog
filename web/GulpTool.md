# gulp-inject
```HTML
<head>
<!-- inject:css -->
<!-- endinject -->
</head>
```

```JS
gulp.src('web/*.html')
    .pipe($.inject(gulp.src('web/js/**/*.js'), { read: false, relative: true}))
    .pipe($.inject(gulp.src('web/css/**/*.css'), { read: false, relative: true}))
    .pipe(gulp.dest('web'))
    .pipe($.size());
```

# useref
Replaces references to non-optimized scripts or stylesheets into a set of HTML files (or any templates/views).
```HTML
<!-- build:<type>(alternate search path) <path> -->
... HTML Markup, list of script / link tags.
<!-- endbuild -->
```

```JS
gulp.src('app/*.html')
        .pipe(assets)
        .pipe(assets.restore())
        .pipe(useref())
        .pipe(gulp.dest('dist'));
```

# gulp-csso
optimize css

```JS
var gulp = require('gulp');
var csso = require('gulp-csso');

gulp.task('default', function() {
    return gulp.src('./main.css')
        .pipe(csso())
        .pipe(gulp.dest('./out'));
});
```

# rev
Static asset revisioning by appending content hash to filenames: unicorn.css → unicorn-098f6bcd.css

```JS
gulp.task('default', function () {
    return gulp.src('src/*.css')
        .pipe(rev())
        .pipe(gulp.dest('dist'));
        });
```

# rev-replace
Rewrite occurences of filenames which have been renamed by gulp-rev