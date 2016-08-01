# Protractor-headless

## Docker

### Dockerfile
`Dockfile` is from [https://github.com/jciolek/docker-protractor-headless/blob/master/Dockerfile](https://github.com/jciolek/docker-protractor-headless/blob/master/Dockerfile)

```
FROM node:slim
MAINTAINER j.ciolek@webnicer.com
WORKDIR /tmp
RUN npm install -g protractor mocha jasmine && \
    webdriver-manager update && \
    apt-get update && \
    apt-get install -y xvfb wget openjdk-7-jre && \
    wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb && \
    dpkg --unpack google-chrome-stable_current_amd64.deb && \
    apt-get install -f -y && \
    apt-get clean && \
    rm google-chrome-stable_current_amd64.deb && \
    mkdir /protractor
ADD protractor.sh /protractor.sh
# Fix for the issue with Selenium, as described here:
# https://github.com/SeleniumHQ/docker-selenium/issues/87
ENV DBUS_SESSION_BUS_ADDRESS=/dev/null
WORKDIR /protractor
ENTRYPOINT ["/protractor.sh"]
```

### Docker Image
- Pull docker image: `docker pull docker pull webnicer/protractor-headless`
- The docker image url [https://hub.docker.com/r/webnicer/protractor-headless/](https://hub.docker.com/r/webnicer/protractor-headless/)

## Usage

### Create `protractor.sh` file
- Make a new file named by `protractor.sh` in `/usr/local/bin/`
- Change the file executable `chmod 755 protractor.sh`
- copy the following content in the file 

```
#!/bin/bash

docker run -it --privileged --rm --net=host -v /dev/shm:/dev/shm -v $(pwd):/protractor webnicer/protractor-headless $@
```

### Check `protractor.sh`
Type `protractor.sh --version` in command line.When the version appear,it's ok.

### Create the pratoractor files 

- `conf.js`

```
// conf.js
exports.config = {
  framework: 'jasmine',
  specs: ['**/**.js'],
  capabilities: {
    browserName: 'chrome'
  },
  jasmineNodeOpts: {
    showColors: true,
  }
};
```

- `test-spec.js`

```
describe('angularjs homepage todo list', function() {
  it('should add a todo', function() {
    browser.get('https://angularjs.org');

    element(by.model('todoList.todoText')).sendKeys('write first protractor test');
    element(by.css('[value="add"]')).click();

    var todoList = element.all(by.repeater('todo in todoList.todos'));
    expect(todoList.count()).toEqual(3);
    expect(todoList.get(2).getText()).toEqual('write first protractor test');

    // You wrote your first test, cross it off the list
    todoList.get(2).element(by.css('input')).click();
    var completedAmount = element.all(by.css('.done-true'));
    expect(completedAmount.count()).toEqual(2);
  });
});
```

### Execute 
- Type `protractor.sh conf.js` in `protractor files` root folder

```
[11:26:01] I/local - Starting selenium standalone server...
[11:26:01] I/launcher - Running 1 instances of WebDriver
[11:26:01] I/local - Selenium standalone server started at http://192.168.8.107:56219/wd/hub
Started
.


1 spec, 0 failures
Finished in 10.044 seconds
[11:26:13] I/local - Shutting down selenium standalone server.
[11:26:13] I/launcher - 0 instance(s) of WebDriver still running
[11:26:13] I/launcher - chrome #01 passed
```
