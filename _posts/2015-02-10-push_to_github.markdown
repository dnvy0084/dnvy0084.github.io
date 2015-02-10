---
layout: post
title:  "Grunt Automation"
date:   2015-02-10 10:00:00
categories: update
---

# [Grunt-Shell plugin]

Grunt를 통해 <small>-[Grunt Build 포스트]-</small> 소스를 관리하다 보면 자동화 된 Task가 필요할 수 있습니다. [Grunt watch plugin]을 이용하면 소스가 변경될 때 마다 어느정도 선에서 정해진 일을 수행 할 수 있는데요. 최종 결과물 파일을 배포 하거나, Git으로 commit 하거나, 수정 log를 남기거나 등등.. 사용자 수 만큼 필요한 일도 다양하여 원하는 일을 모두 해줄 수는 없습니다. 그럴 때 유용한 플러그인이 `grunt-shell` 입니다.

`grunt-shell`은 grunt task에 command line 명령어를 추가할 수 있게 해주는데요, 사용방법은 여느 플러그인과 같습니다. 

`npm install grunt-shell --save-dev` 를 이용해 프로젝트 폴더에 `install`하고

`grunt.loadNpmTasks( 'grunt-shell' );` loadNpmTasks로 플러그인을 로드 하면 준비가 끝납니다.


`grunt-shell`은 nodejs의 [child process]를 이용하여 구현되어 있는데요, 간단한 shell command는 child process에 대한 지식 없이도 사용할 수 있습니다. 


```json
shell: 
{
    options: {
        stderr: false
    },

    target: {
        command: 'dir'
    }
}
```

`grunt.initConfig`에 위와 같이 `shell` 실행 옵션을 추가하고 watch나 default task에 'shell'을 추가하여 실행하면 다음과 같이 dir 명령이 실행된 모습을 볼 수 있습니다. 

![alt text][1]

아래처럼 `&`로 구분하여 여러개의 명령을 실행 할 수도 있습니다. src폴더에서 변경사항이 생기면 watch를 통해 git commit과 push를 실행하여 자동으로 github: gh-pages에 deploy를 하고 있습니다. 

```json
shell: 
{
    options: {
        stdout: true,
        stderr: true
    },

    target: {
        command: [ 
            'git add .',
            'git commit -m "commited by grunt watch at <%= grunt.template.today("yyyy-mm-dd, hh:MM:ss TT") %>"',
            'git push origin gh-pages'
        ].join( '&' )
    }
}
```

Grunt-Shell을 이용하면 github push 뿐만 아니라 ftp 업로드, 로그 작성, Dev 서버 배포 등 사용자가 원하는 거의 모든 일을 자동화 할 수 있습니다. 다음은 Gruntfile.js code입니다. 

```javascript
module.exports = function(grunt) {

    // Project configuration.
    grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        // uglify 설정. 
        uglify: {
            options: {
                banner: '/*! <%= pkg.name %> [<%= pkg.version %>] <%= grunt.template.today("yyyy-mm-dd, hh:MM:ss TT") %> */\n'
            },
            build: {
                src: 'build/<%= pkg.name %>.js',
                dest: 'js/<%= pkg.name %>.min.js'
            }
        },

		// shell 설정. 
        shell: 
        {
            options: {
                stdout: true,
                stderr: true
            },

            target: {
                command: [ 
                    'git add .',
                    'git commit -m "commited by grunt watch at <%= grunt.template.today("yyyy-mm-dd, hh:MM:ss TT") %>"',
                    'git push origin gh-pages'
                ].join( '&' )
            }
        },

        // concat 설정. 
        concat: {
            basic: {
                src: [
                    "src/*.js" 
                ],

                dest: "build/<%= pkg.name %>.js" 
            }
        },

        watch: {
            scripts: {
                files: ['src/*.js'],
                tasks: ['concat', 'uglify', 'shell']
            },
        }
    });

    // Load the plugin that provides the "uglify" task.
    grunt.loadNpmTasks('grunt-contrib-uglify');

    // Load the plugin that provides the "concat" task.
    grunt.loadNpmTasks( 'grunt-contrib-concat' );

    // load watch
    grunt.loadNpmTasks( 'grunt-contrib-watch' );

    // load shell command plugins
    grunt.loadNpmTasks( 'grunt-shell' );

    // Default task(s)
    grunt.registerTask('default', ['concat', 'uglify']);

};
```


[Grunt Build 포스트]: http://dnvy0084.github.io/update/2015/02/05/grunt_build.html
[Shell plugin]: https://github.com/sindresorhus/grunt-shell
[child process]: http://nodejs.org/api/child_process.html
[Grunt watch plugin]: https://github.com/gruntjs/grunt-contrib-watch

[1]: /raw/grunt-shell-dir.png "grunt shell result"