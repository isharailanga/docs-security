﻿---
title: General Recommendations for React Secure Coding
category: security-guidelines
published: January 26, 2021
---

# General Recommendations for React Secure Coding

<p class="doc-info">Published: January 26, 2021</p>
<p class="doc-info">Version: 1.0.0</p>
---

## Revision History 


| Version | Release Date | Contributors / Authors | Summary of Changes |
| ------- | ------------ | ---------------------- | ------------------ |
| 1.0.0   | January 26, 2021 | [Anshajanth Yoganathan](https://wso2.com/about/team/anshajanth-yoganathan/) | Initial version |


## Introduction
This document summarizes the [WSO2 Secure Coding Guidelines]({{base_path}}/security-guidelines/secure-engineering-guidelines/secure-coding-guidlines/introduction/) that should be followed by WSO2 engineers while developing **React** based WSO2 products, as well as applications used within the organization. 

The purpose of this document is to increase the security awareness and make sure the products and the applications developed by WSO2 are inherently secure, by making sure security best practices are followed throughout the Software Development Life Cycle. 


## Data Binding
Untrusted user inputs must be handled by react's default data binding with curly brackets. because react’s data binding feature has capability to neutralize the html tags by default. So malicious user's input such as xss script will not be executed on the client side[^1].

!!! success check done "Example Correct Usage"
    ```js
    function Wso2(){
      Const input="hello <img src=x onerror=alert(0)>";
      Return (<dev>{input}</dev>);}
    ```

Note above mentioned protection only occurs when rendering user input's data as text content. Likewise this security measurement could be bypassable when rendering the untrusted user input as a html attribute.

!!! bug error "Example Incorrect Usage"
    ```js
    function wso2(){
      Const input="data:text/html,<script>alert(1)</script>";
      Return (<iframe src={input}></iframe>);
    }
    ```

To avoid these unwanted malicious scripts execution, all untrusted user inputs data must be sanitized before put in to the html attribute.

!!! success check done "Example Correct Usage"
    ```js
    Import purify from “dompurify”;

    function wso2(){
      Const input="data:text/html,<script>alert(1)</script>";
      Return (<iframe src="DOMPurify.sanitize(input)"></iframe>);
    }
    ```


## Url Handling
Urls also can contain dynamic script via "javascript protocol urls". So highly recommended to validate the URLs before using it. When using a validation to an url that should be checked whether that the link has http or https component to avoid javascript based script injection[^2].

!!! bug error "Example Incorrect Usage"
    ```js
    // Classic XSS via anchor tag href attribute.
    <a href="javascript: alert(1)">Click me!</a>
    ```

!!! success check done "Example Correct Usage"
    ```js
    isSafe(dangerousURL, text) {
      const url = URL(dangerousURL, {})
      if (url.protocol === 'http:') return true
      if (url.protocol === 'https:') return true    

      return false
    }
    ```


## HTML Rendering
It is possible to insert html tags directly into rendered DOM nodes using the DOM element's attribute called `dangerouslySetInnerHTML`. If any required situation to use the above mentioned attribute into the wso2 product components that must be contained properly sanitization[^3].

!!! bug error "Example Incorrect Usage"
    ```js
    function wso2(){
      Const input="hello <img src=x onerror=alert(0)>";
      Return (<dev dangerouslySetInnerHTML={{_html:input}}></dev>);
    }
    ```

To sanitize the input can use a sanitization library like `DOMPurify`[^4] on any values before placing them into the `dangerouslySetInnerHTML`.

!!! success check done "Example Correct Usage"
    ```js
    import purify from "dompurify";

    function wso2(){
      Const input="hello <img src=x onerror=alert(0)>";
      Return (<dev dangerouslySetInnerHTML={{_html:DOMPurify.sanitize(input)}}></dev>);
    }
    ```


## Direct DOM Access
Accessing the DOM to insert the content into DOM nodes directly should be avoided. So it is highly recommended to apply a proper sanitization when `findDomNode()` and `createRef` to access rendered DOM elements directly to inject content via innerHTML and similar properties.

!!! bug error "Example Incorrect Usage"
    ```js
    class App extends React.Component {   
      changeMe(){
        let input = "<img src=x onerror=alert(0)>";
        ReactDOM.findDOMNode(document.getElementById("wso2")).innerHTML= input;
      }

      render() {
        return (
          <dev>
            <button onClick={this.changeMe}>change me</button><br></br>
            <dev id="wso2">Welcome to XSS Test</dev>
          </dev>
        )
      }
    }
    ```

!!! success check done "Example Correct Usage"
    ```js
    class App extends React.Component {   
      changeMe(){
        let input = "<img src=x onerror=alert(0)>";  ReactDOM.findDOMNode(document.getElementById("wso2")).innerHTML=DOMPurify.sanitize(input);
      }

      render()
      {
        return (
          <dev>
            <button onClick={this.changeMe}>change me</button><br></br>
            <dev id="wso2">Welcome to XSS Test</dev>
          </dev>
        )
      }
    }
    ```


## Server Side Rendering

### JSON
Sometimes when rendering the initial state in server side, that dangerously generates a document variable from a JSON string. This is risky because `JSON.stringify()` will blindly turn any data that you give it into a string (so long as it is valid JSON) which will be rerendered in the page. If fields that untrusted users edit and inject malicious scripts.

!!! bug error "Example Incorrect Usage"
    ```js
    <script>
      window._PRELOADED_STATE_ = ${JSON.stringify(untrusted_JSON_data)}
    </script>
    ```

To fix this vulnerability when serializing state on the server to be sent to the client, it must be serialized in a way to sanitize the HTML tags. It is highly recommended to use a library like `serialize-javascript`[^5] to avoid unnecessary HTML tag renders[^6].

!!! success check done "Example Correct Usage"
    ```js
    import purify from "dompurify";

    <script>
      window._PRELOADED_STATE_ = ${serialize(untrusted_JSON_data,{isJSON:true})}
    </script>
    ```


## Third Party Dependencies
Before using any 3rd party npm packages or libraries on the wso2 products, highly recommended to ensure that the above mentioned vulnerable code components are not used in those dependencies. Also these 3rd party dependencies must not contain any known vulnerabilities. To detect vulnerabilities on 3rd party components can use tools like `nmp audit`[^7].


## References
[^1]: [https://pragmaticwebsecurity.com/articles/spasecurity/react-xss-part1.html](https://pragmaticwebsecurity.com/articles/spasecurity/react-xss-part1.html)
[^2]: [https://medium.com/javascript-security/avoiding-xss-in-react-is-still-hard-d2b5c7ad9412](https://medium.com/javascript-security/avoiding-xss-in-react-is-still-hard-d2b5c7ad9412)
[^3]: [https://pragmaticwebsecurity.com/articles/spasecurity/react-xss-part2.html](https://pragmaticwebsecurity.com/articles/spasecurity/react-xss-part2.html)
[^4]: [https://github.com/cure53/DOMPurify](https://github.com/cure53/DOMPurify)
[^5]: [https://www.npmjs.com/package/serialize-javascript](https://www.npmjs.com/package/serialize-javascript)
[^6]: [https://www.veracode.com/blog/secure-development/3-security-pitfalls-every-react-developer-should-know](https://www.veracode.com/blog/secure-development/3-security-pitfalls-every-react-developer-should-know)
[^7]: [https://docs.npmjs.com/cli/v6/commands/npm-audit](https://docs.npmjs.com/cli/v6/commands/npm-audit)
