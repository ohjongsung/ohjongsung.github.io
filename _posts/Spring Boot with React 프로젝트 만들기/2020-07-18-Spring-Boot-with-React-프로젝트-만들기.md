이전 회사에서 Spring Boot 랑 React.js 가 한 프로젝트에 묶여 있는 환경에서 개발한 경험이 있다. 이미 갖춰진 프로젝트에서 기능을 추가하는 수준이라 React 만 공부해서 개발했었다. 이번에 내가 직접 프로젝트를 구성해보니, 생각처럼 쉽지가 않았다. 이전 프로젝트 구조가 개발과 운영까지 생각해서 빌드 구조가 잘 짜여져 있었는데, 인터넷 검색으로는 그런 수준의 글을 찾지 못했다. 프론트 개발자가 아니라서 조금 어설프지만 나름 원하는 모양새는 갖춰서 정리한다.

## Spring Boot Project 생성

#### dependency

spring-boot-starter-web 모듈만 있으면 충분하다.

```xml
<dependencies>
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
	<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <scope>runtime</scope>
      <optional>true</optional>
  </dependency>
</dependencies>
```

#### HomeController 생성

* '/' 로 접근하면 static/index.html 로 매핑한다. 실제로 html 파일을 만들 필요없다.
* front-end 와 연동 예제로 현재 시간을 내려주는 'api/now' endpoint 생성

```java
@Controller
public class HomeController {
	
	@GetMapping("/")
	public String index() {
		return "index.html";
	}

	@GetMapping("api/now")
	public @ResponseBody String now() {
		return "안녕하세요. 현재 서버시간은 "+new Date() +"입니다. \n";
	}
}
```

#### pom.xml

maven profile 을 dev 와 prod 를 나눈다. 실제 배포할 때, prod 로 실행해서 frontend source 를 build 해서 resource/static 폴더에 넣어 준다.

```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
				<build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    </profile>
    <profile>
        <id>prod</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>com.github.eirslett</groupId>
                    <artifactId>frontend-maven-plugin</artifactId>
                    <version>1.9.1</version>
                    <configuration>
                        <workingDirectory>frontend</workingDirectory>
                        <installDirectory>target</installDirectory>
                    </configuration>
                    <executions>
                        <execution>
                            <id>install node and npm</id>
                            <goals>
                                <goal>install-node-and-npm</goal>
                            </goals>
                            <configuration>
                                <nodeVersion>v12.18.2</nodeVersion>
                                <npmVersion>6.14.5</npmVersion>
                            </configuration>
                        </execution>
                        <execution>
                            <id>npm install</id>
                            <goals>
                                <goal>npm</goal>
                            </goals>
                            <configuration>
                                <arguments>install</arguments>
                            </configuration>
                        </execution>
                        <execution>
                            <id>npm run build</id>
                            <goals>
                                <goal>npm</goal>
                            </goals>
                            <configuration>
                                <arguments>run build</arguments>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <executions>
                        <execution>
                            <id>copy-resources</id>
                            <phase>process-classes</phase>
                            <goals>
                                <goal>copy-resources</goal>
                            </goals>
                            <configuration>
                                <outputDirectory>${basedir}/target/classes/static</outputDirectory>
                                <resources>
                                    <resource>
                                        <directory>frontend/public</directory>
                                    </resource>
                                </resources>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

#### .gitignore

.gitignore 도 미리 만들어놓자.

```
HELP.md
target/
!.mvn/wrapper/maven-wrapper.jar

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/

### VS Code ###
.vscode/

### frontend ###
frontend/public/
frontend/node_modules/
frontend/node/
frontend/.env
frontend/.cache/
frontend/yarn-error.log
frontend/analysis
```

## React.js Project 생성

#### node.js 설치

[node.js](https://nodejs.org/ko/)
node.js 를 설치하고 버전확인

```bash
node -v
npm —v
```

#### React.js Project 생성

- create-react-app 으로 react.js project 를 만들면 local git repository 생성된다. spring boot 와 함께 형상관리해야 하므로 react.js project 의 .git folder 를 삭제한다.
- 이후 작업은 모두 frontend 폴더에서 이루어진다.

```shell
npx create-react-app frontend
cd frontend

# 실행 확인
npm start or yarn start
```

#### Adding Bootstrap & react library

- ui framework antd 설치
- react library 추가

```bash
npm i antd @ant-design/icons --save
npm i react react-dom axios --save
```

#### App.js 수정

- back-end api 와 연동 예제
- ui framework 활용 예제

```jsx
import React, {useEffect, useState} from 'react';
import axios from 'axios';
import {Layout, Menu } from 'antd';
import {UserOutlined, FileDoneOutlined, TeamOutlined} from '@ant-design/icons';
import './App.css';

const {SubMenu} = Menu;
const {Header, Content, Sider, Footer} = Layout;

function App() {
    const [message, setMessage] = useState('오늘');

    useEffect(() => {
        axios.get('/api/now')
            .then((response) => setMessage(response.data))
    })

    return (
        <Layout style={{ minHeight: '100vh' }}>
            <Sider style={{
                overflow: 'auto',
                height: '100vh',
                position: 'fixed',
                left: 0,
            }}>
                <div className="logo" />
                <Menu
                    theme="dark"
                    mode="inline"
                    defaultSelectedKeys={['1']}
                    defaultOpenKeys={['sub1']}
                >
                    <SubMenu
                        key="sub1"
                        title={<span><UserOutlined /><span>User</span></span>}
                    >
                        <Menu.Item key="3">Tom</Menu.Item>
                        <Menu.Item key="4">Bill</Menu.Item>
                        <Menu.Item key="5">Alex</Menu.Item>
                    </SubMenu>
                    <SubMenu
                        key="sub2"
                        title={<span><TeamOutlined /><span>Team</span></span>}
                    >
                        <Menu.Item key="6">Team 1</Menu.Item>
                        <Menu.Item key="8">Team 2</Menu.Item>
                    </SubMenu>
                    <Menu.Item key="9">
                        <FileDoneOutlined/>
                        <span>File</span>
                    </Menu.Item>
                </Menu>
            </Sider>
            <Layout className="site-layout" style={{ marginLeft: 200 }}>
                <Header className="site-layout-background" style={{ padding: 0 }} >
                    <Menu mode="horizontal" defaultSelectedKeys={['2']}>
                        <Menu.Item key="1">nav 1</Menu.Item>
                        <Menu.Item key="2">nav 2</Menu.Item>
                        <Menu.Item key="3">nav 3</Menu.Item>
                    </Menu>
                </Header>
                <Content style={{ margin: '24px 16px 0', overflow: 'initial' }}>
                    <div className="site-layout-background" style={{ padding: 24, textAlign: 'center' }}>
                        <br />
                        {message}
                        <br />
                    </div>
                </Content>
                <Footer style={{ textAlign: 'center' }}>Ant Design ©2018 Created by Ant UED</Footer>
            </Layout>
        </Layout>
    );
}

export default App;
```

#### App.css 수정

```css
@import '~antd/dist/antd.css';

.logo {
    height: 32px;
    background: rgba(255, 255, 255, 0.2);
    margin: 16px;
}

.site-layout .site-layout-background {
    background: #fff;
}

[data-theme="dark"] .site-layout .site-layout-background {
    background: #141414;
}
```

#### static folder 를 만들고 index.html 추가

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title><%= htmlWebpackPlugin.options.title %></title>
    <link rel="canonical" href="<%= htmlWebpackPlugin.options.url %>" />
    <meta
            name="description"
            content="<%= htmlWebpackPlugin.options.description %>"
    />
</head>
<body>
    <!--이 ID 값은 index.js 의 ReactDOM render 시에 사용 됩니다.-->
    <div id="root"></div>
</body>
</html>
```

####**Webpack & Bable 설치**####

```bash
# i == install
# --save-dev == -D ~> devDependencies 에 설치
# --save == -S ~> dependencies 에 설치
# 여러 패키지 한 번에 설치하고 싶으면 패키지명 사이에 한 칸 space로 구분
npm i webpack --save-dev
npm i webpack-cli --save-dev
npm i webpack-merge --save-dev
npm i clean-webpack-plugin --save-dev
npm i webpack-dev-server --save-dev
npm i html-webpack-plugin html-loader --save-dev
npm i @babel/cli babel-loader @babel/core @babel/preset-env @babel/preset-react --save-dev
npm i cross-env --save-dev
```

* .babelrc 파일 추가

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

####**Webpack 설정**####

**webpack.common.js 추가 및 설정**

```jsx
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const isProd = process.env.NODE_ENV === 'production'

const config = {
    entry: {
        index: path.resolve(__dirname, 'src/index.js')
    },
    output: {
        path: path.resolve(__dirname, 'public'),
        publicPath: '/',
        filename: '[name]-[hash].js',
        chunkFilename: 'chunk-[chunkhash].js',
    },
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                exclude: [path.resolve(__dirname, 'node_modules')],
                use: {
                    loader: "babel-loader"
                }
            },
            {
                test: /\.css$/,
                use: ["style-loader", "css-loader"],
            },
            {
                test: /\.(png|svg|jpe?g|gif)$/,
                loader:'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin({cleanAfterEveryBuildPatterns: ['public']}),
        new HtmlWebpackPlugin({
            minify: isProd
                ? true
                : {
                    collapseWhitespace: true,
                    removeComments: true,
                    useShortDoctype: true,
                    minifyCSS: true,
                },
            template: './static/index.html',
            title: 'Spring Boot + React.js',
            description: `Spring Boot + React.js Example`,
            url: 'https://ohjongsung.io',
        })
    ]
};

module.exports = config
```

**webpack.dev.js 추가 및 설정**

```jsx
const { merge } = require('webpack-merge')
const common = require('./webpack.common.js')
const path = require('path')

const config =  {
    mode: 'development',
    devtool: 'inline-source-map',
    devServer: {
        contentBase: path.resolve(__dirname, 'public'),
        historyApiFallback: true,
        compress: true,
        open: true,
        port: 3000,
        host: 'localhost',
        // dev 환경에서 cors 대응
        proxy: {
            '/api': 'http://localhost:8080',
        }
    },
}

module.exports = merge(common, config)
```

**webpack.prod.js 추가 및 설정**

```jsx
const { merge } = require('webpack-merge')
const common = require('./webpack.common.js')

const config = {
    mode: 'production',
    devtool: 'source-map',
    optimization: {
        runtimeChunk: {
            name: "runtime"
        },
        splitChunks: {
            name: "vendor",
            chunks: "all"
        },
    }
}

module.exports = merge(common, config)
```

**.npmignore 추가**

```yaml
support
test
examples
.gitignore
History.md
```

**package.json 수정**

```
"scripts": {
  "start": "webpack-dev-server --progress --colors --config webpack.dev.js",
  "build": "cross-env NODE_ENV=production webpack -p --config webpack.prod.js"
}

# 개발할 때, npm start
# 배포할 때, npm run build
```

## 실행

**개발할때**

- front & back 을 각각 실행시켜야 한다.
- react.js 코드를 수정하면 auto reload 된다.

```bash
# IDE 에서 실행하거나
mvn spring-boot:run

cd frontend
npm start
```

**배포할때**

- target/classes/static 폴더에 frontend source 가 빌드되어 담겨지고 jar 에 포함된다.

```bash
# 빌드
mvn clean package -P prod

# 실행
java -jar xxx-0.0.1-SNAPSHOT.jar
```
