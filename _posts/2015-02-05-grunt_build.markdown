---
layout: post
title:  "Grunt Build"
date:   2015-02-05 10:00:00
categories: update
---

# Grunt

Grunt는 **"a task-based command line build tool for JavaScript projects"** 입니다. javascript 프로젝트 관리툴로 이것저것 유용한 플러그인도 많고 쓰면 엄청 편하다(하지만 command line이니 감수해야되) 라고 하는데, concat과 uglify 플러그인만 사용해도 충분한것 같습니다. 두개만 사용할 수 있도록 기본 세팅 정도만 포스팅 하겠습니다. windows 기반입니다. 

* 전역 설정

	Grunt는 [nodejs] 로 작동합니다. 당연히 nodejs 부터 [다운로드] 받아 인스톨 합니다. 완료 후 환경 변수에도 등록하여 global로 사용할 수 있도록 합니다. command line 창을 열어 설치된 폴더가 아닌 다른 폴더에서 

	`node --version` 

	했을 때 v0.xx.xx 로 출력되면 정상입니다.

	Grunt는 grunt-cli 라는 nodejs package를 통해 전역적으로 실행을 할 수 있지만, 설정이나 사용 플러그인 세팅은 프로젝트 폴더 각각 따로 설치해야 합니다. 우선 grunt-cli를 npm( nodejs package manager )을 통해 설치합니다. 

	cmd창에서 `npm install -g grunt-cli` 라고 입력하면 설치 됩니다. `-g`는 global입니다. 설치 후 `grunt --version`으로 버전 확인할 수 있습니다. 


	```
	grunt-cli v0.1.13
	grunt v0.4.5
	```
	위처럼 출력되면 정상입니다. 여기까지가 Grunt 사용을 위한 전역 설정입니다. 


* 개별 프로젝트 
	
	Grunt를 사용할 프로젝트 폴더에서 `npm init` 을 입력하면 패키지 세팅에 관한 세부 항목들을 설정할 수 있는데요, 그냥 엔터로 넘겨도 상관없습니다. 
	
	![alt text][1]
	
	모두 설정하면 폴더에 package.json 파일이 생성됩니다. 패키지 명이나 버전, 설명등이 포함되며 사용 플러그인 정보도 들어있습니다. 기존 프로젝트 폴더에 git 정보가 있다면 repository에 자동으로 입력됩니다. 이렇게 package.json 파일을 생성하고 나면 사용할 플러그인을 인스톨해야 되는데요, 위에서 말한 concat과 uglify를 설치할 차례입니다. 

	concat은 여러개의 js파일을 하나로 합쳐주고, uglify는 formatted된 code를 한줄로 만들어 주는 플러그인입니다. 흔히 말하는 source.min.js를 생성해줍니다. 

	플러그인 설치는 `npm install 원하는 플러그인 명 --save-dev` 로 설치할 수 있습니다. `--save-dev` 옵션은 플러그인 설치와 함께 package.json에 플러그인 정보도 같이 기록합니다. 


	```
	npm install grunt-contrib-concat --save-dev
	npm install grunt-contrib-uglify --save-dev
	```
	
	위와 같이 두개 플러그인을 설치하고 package.json에 

	```
	"devDependencies": {
	    "grunt": "^0.4.5",
	    "grunt-contrib-concat": "^0.5.0",
	    "grunt-contrib-uglify": "^0.7.0"
	}
	```
	위와 같이 추가되어 있다면 정상 설치된겁니다. `"grunt": "^0.4.5"`는 기본 플러그인이라 자동으로 설치됩니다. 마지막으로 빌드 경로나 플러그인 사용 순서를 적용할 Gruntfile.js를 생성해야 합니다. grunt-init 같은 template 플러그인이 있다고 하는데 그냥 새파일 만들고 Gruntfile.js로 저장해서 아래 코드를 붙여넣기 해도 됩니다. (grunt-init template 생성이 잘 안되요..)


	```javascript
	module.exports = function(grunt) {

		// Project configuration.
		grunt.initConfig({
			pkg: grunt.file.readJSON('package.json'),
			// uglify 설정. 
			uglify: {
				options: {
					banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
				},
				build: {
					src: 'build/<%= pkg.name %>.js',
					dest: 'build/<%= pkg.name %>.min.js'
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
			}
		});

		// Load the plugin that provides the "uglify" task.
		grunt.loadNpmTasks('grunt-contrib-uglify');

		// Load the plugin that provides the "concat" task.
		grunt.loadNpmTasks( 'grunt-contrib-concat');

		// Default task(s)
		grunt.registerTask('default', ['concat', 'uglify']);

	};
	```

* Build
	
	모든 설정이 완료되었다면 빌드 할일 만 남았는데요, 빌드는 cmd 창에서 `grunt`라고 입력하면 됩니다. 위처럼 설정해 놓았다면 build 폴더에 패키지명.js와 패키지명.min.js가 생성됩니다. 그런데 매번 수정할 때마다 `grunt`로 build하는것도 만만한 작업은 아닌것 같은데요, [watch] 플러그인을 이용하면 파일이 변경될때마다 자동으로 build할 수 있도록 해줍니다. 

	```javascript
	watch: {
		scripts: {
			files: ['**/*.js'],
			tasks: ['concat']
		},
	}
	```

	```javascript
	grunt.loadNpmTasks('grunt-contrib-watch');
	```

	Gruntfile.js에 위 코드를 추가하고 cmd창 프로젝트 폴더에서 


	```
	grunt watch
	```
	로 실행해 놓으면 js파일이 변경될때 마다 `concat`을 실행하게 됩니다. `['concat', 'uglify']`라고 설정하면 concat후 uglify까지 실행합니다. 여기까지 해놓았으면 이제 남은건 불꽃코딩 밖에 없네요. 


[nodejs]: http://nodejs.org/
[다운로드]: http://nodejs.org/download/
[watch]: https://github.com/gruntjs/grunt-contrib-watch

[1]: /raw/grunt-init.jpg "grunt package setting"