셋팅 순서

윈도우의 경우 mvn이 안먹힐 경우 .\mvnw 으로 변경하셔서 돌리시면 됩니다. 터미널에서 m 입력후 tab을 누르시면 자동완성됩니다.

1. https://start.spring.io
위 사이트 방문 셋팅 후 dependency에 web추가

2. pom.xml설정

<code>
    
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <groupId>no.kantega</groupId>
        <artifactId>spring-and-react</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <packaging>jar</packaging>

        <name>spring-and-react</name>
        <description>Demo project for Spring Boot</description>

        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.0.1.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>

        <properties>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
            <java.version>1.8</java.version>
        </properties>

        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>

        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    </project>

</code>

3. Controller설정

--참고 vscode를 사용시 auto import는 alt + shift + O이다.

<code>
    
    package no.kantega.springandreact;

    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;

    import java.util.Date;

    @RestController
    public class HelloController {
        @GetMapping("/api/hello")
        public String hello() {
            return "Hello, the time at the server is now " + new Date() + "\n";
        }
    }
    
</code>

4. mvn spring-boot:run실행

5. 리엑트를 추가시킨다

npx create-react-app frontend

6. http-proxy-middlewar를 추가한다. -개발시 필요함.

기존에 spring-boot는 8080 port를 사용한다면 react의 경우 3000 port를 사용함. 개발시 3000 port에서 8080 port로 연결시켜주는 역활을 할 예정

cd frontend 

npm install --save http-proxy-middleware

7. 리엑트 실행

실행위치는 frontend 폴더 안이어야합니다.

npm start

초기 실행시에 시간이 걸릴 수 있습니다.

8. proxy연결

frontend/src/ setupProxy.js추가

내용은

<code>
    
    const proxy = require('http-proxy-middleware')

    module.exports = function(app) {
        app.use(proxy('/', { target: 'http://localhost:8080/' }));
    }
    
</code>

npm start

연결 테스트 : curl http://localhost:3000/api/hello

이제 연결됬는지 확인

frontend/src/App.js파일을 열어 다음과 같이 수정

<code>
    
    import React, {Component} from 'react';
    import logo from './logo.svg';
    import './App.css';

    class App extends Component {

        state = {};

        componentDidMount() {
            setInterval(this.hello, 250);
        }

        hello = () => {
            fetch('/api/hello')
                .then(response => response.text())
                .then(message => {
                    this.setState({message: message});
                });
        };

        render() {
            return (
                <div className="App">
                    <header className="App-header">
                        <img src={logo} className="App-logo" alt="logo"/>
                        <h1 className="App-title">{this.state.message}</h1>
                    </header>
                    <p className="App-intro">
                        To get started, edit <code>src/App.js</code> and save to reload.
                    </p>
                </div>
            );
        }
    }

    export default App;
    
</code>

화면이 제대로 바뀌면 성공

9. 배포용 패키지 생성

pom.xml 안에 /build/plugins에 기존 plugin아래 추가

<code>
    
    <plugin>
        <groupId>com.github.eirslett</groupId>
        <artifactId>frontend-maven-plugin</artifactId>
        <version>1.6</version>
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
                    <nodeVersion>v8.9.4</nodeVersion>
                    <npmVersion>5.6.0</npmVersion>
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
    
</code>

추가후 

기존 mvn spring-boot:run 했던 터미널창에서 ctrl + C 연타하여 서버 다운후 재가동.

mvn clean install

위 과정은 react파일을 spring boot에 포함시키기기 위해 빌드했습니다.

10. 빌드된 파일 jar파일에 포함하기

pom.xml 안에 /build/plugins에 기존 plugin아래 추가

<code>

    <plugin>
        <artifactId>maven-antrun-plugin</artifactId>
        <executions>
            <execution>
                <phase>generate-resources</phase>
                <configuration>
                    <target>
                        <copy todir="${project.build.directory}/classes/public">
                            <fileset dir="${project.basedir}/frontend/build"/>
                        </copy>
                    </target>
                </configuration>
                <goals>
                    <goal>run</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    
</code>

추가후 

기존 mvn spring-boot:run 했던 터미널창에서 ctrl + C 연타하여 서버 다운후 재가동.

mvn clean install

위 과정을 걸치면 projectDir/target/ jar파일이 생성됨.

jar파일을 실행해봅시다.

cd target/

java -jar .\reactwithspringboot-0.0.1-SNAPSHOT.jar

이렇게 되면 끝

확인은 localhost:8080
