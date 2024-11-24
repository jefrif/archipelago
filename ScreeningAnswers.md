
# Screening Answer
### Jefri Ferli
#
This document can be obtained from: `git clone https://github.com/jefrif/archipelago.git`
 
### Basic Questions
_To design the logic for the process, I would use_:
1. UML Use case diagram to model the system's functional context. What are the functions the application provide that its actor expect to have.
2. Web Site Map diagram to model the flow of pages during user interaction
3. Balsamiq to make the sketch of the UI layout

_Additionally, for the more complex system, I will use_: 
- UML sequence diagram to model the interaction between the application and its actor for each use case
- UML activity diagram to model the control and data flow thru the processes that are performed by or performed on the actors or the internal objects.

_The technologies I would use_:

- **Backend**: Go based REST API server with Gin framework or Java servlet based JakartaEE Tomcat server hosted on AWS EC2 or other local VPS.
- **Frontend**: Angular 15+ with PrimeNG framework or React with PrimeReact framework or Vue with PrimeVue. All Prime frameworks are used for components, templates, and simplicity.
- **Data store**: Use Redis and Elasticsearch. Both can complement each other by combining their strengths. Redis is an in-memory database known for its speed and efficient caching capabilities, while Elasticsearch is a distributed search and analytics engine designed for indexing, querying, and analyzing large datasets. Use Redis as a staging area to preprocess or temporarily store data before sending it to Elasticsearch for indexing.
This is especially useful in systems that receive a high volume of data in real-time. Redis can act as a message queue using data structures like Lists or Streams to batch and send data to Elasticsearch in bulk. This helps optimize Elasticsearch's performance by reducing the number of individual indexing operations.

_The data store structure_:
- User: Hashes and Sets
Each user can be stored as a Redis hash, with the id as the key and fields (name, type) as hash fields.  
If we need to group users by type, you can use a Set or Sorted Set in addition to Hashes.
- Picture: Metadata (Hash):
Store id, title, and updated_time, user_id, reviewer_id in a Redis Hash, with the id as part of the key for uniqueness.  
Example: photo:`{id}`:meta  
Picture Content (String):  
Store the binary content of the picture as a Redis String.  
Example: photo:`{id}`:content

### Database Questions
#### Level 1 
```
SELECT COUNT(*) FROM Customers WHERE Country = 'GERMANY';
```
#### Level 2
```
SELECT COUNT(CustomerID) AS CustCount, Country
  FROM Customers GROUP BY Country  HAVING COUNT(CustomerID) >= 5
ORDER BY COUNT(CustomerID) DESC;
```
#### Level 3
```
SELECT CustomerName, COUNT(CustomerName) AS OrderCount, MIN(OrderDate) AS FirstOrder, MAX(OrderDate) AS LastOrder FROM Customers, Orders  WHERE Orders.CustomerID = Customers.CustomerID 
GROUP BY CustomerName;
```

### JavaScript/TypeScript Questions
#### Level 1 
```
function isWhitespace(char) {
  return /\s/.test(char);
}

function titleCase(str) {
  let res = "";
  let i = 0;
  while (i < str.length) {
    while (i < str.length && isWhitespace(str[i])) {
      res += str[i];
      i++;
    }

    let first = i;
    while (i < str.length && !isWhitespace(str[i])) {
      if (first === i) {
        res += str[i].toUpperCase();
      } else {
        res += str[i].toLowerCase();
      }
      i++;
    }
  }

  return res;
}

console.log(titleCase("I am a little tea pot"));
console.log(titleCase("sHoRt AnD sToUt"));
console.log(titleCase("SHORT AND STOUT"));
```
#### Level 2
```
function delay(ms) {
  const promise = new Promise((resolve, reject) => {
    // Perform an async task
    setTimeout(() => {
      let success = true;
      if (success) {
        resolve(ms / 1000);
      } else {
        reject("Failed");
      }
    }, ms);
  });
  
  return promise;
}

delay(3000).then((sec) => alert(`runs after ${sec} seconds`));
```
#### Level 2.5
```
function fetchData(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (!url) {
        reject("URL is required");
      } else {
        resolve(`Data from ${url}`);
      }
    }, 1000);
  });
}

function processData(data) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (!data) {
        reject("Data is required");
      } else {
        resolve(data.toUpperCase());
      }
    }, 1000);
  });
}

async function fetchAndProcessData() {
  try {
    console.log("Fetching data...");
    const data = await fetchData('urlx');
    console.log("Success:", data);
    const result = await processData(data);
    console.log("Success2:", result);
  } catch (error) {
    console.error("Error:", error); // Handle errors
  } finally {
    console.log("Operation complete.");
  }
}

fetchAndProcessData();
```
#### Level 3-4
**socket-server.ts**
```
import { WebSocketServer } from 'ws';

const PORT = 8080;
const server = new WebSocketServer({ port: PORT });

server.on('connection', (ws) => {
    console.log('Client connected');

    ws.on('message', (message) => {
        console.log(`Received: ${message}`);
        server.clients.forEach((client) => {
            if (client.readyState === ws.OPEN) {
                client.send(message);
            }
        });
    });

    ws.on('close', () => {
        console.log('Client disconnected');
    });
});

console.log(`WebSocket server is running on ws://localhost:${PORT}`);
```
**App.vue**
```
<template>
  <div id="app">
    <h1>Real-Time Chat</h1>
    <div class="chat-box">
      <div v-for="(msg, index) in messages" :key="index" class="message">
        {{ msg }}
      </div>
    </div>
    <div class="input-box">
      <input
        type="text"
        v-model="message"
        @keyup.enter="sendMessage"
        placeholder="Type a message..."
      />
      <button @click="sendMessage">Send</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive, onMounted, onBeforeUnmount } from "vue";

export default defineComponent({
  name: "App",
  setup() {
    const state = reactive({
      ws: null as WebSocket | null,
      messages: [] as string[],
      message: "",
    });

    const sendMessage = () => {
      if (state.message.trim() && state.ws) {
        state.ws.send(state.message);
        state.message = "";
      }
    };

    onMounted(() => {
      state.ws = new WebSocket("ws://localhost:8080");
      state.ws.onmessage = (event) => {
        state.messages.push(event.data);
      };
    });

    onBeforeUnmount(() => {
      if (state.ws) {
        state.ws.close();
      }
    });

    return {
      ...state,
      sendMessage,
    };
  },
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  margin: 0 auto;
  max-width: 600px;
}
.chat-box {
  border: 1px solid #ccc;
  height: 300px;
  overflow-y: auto;
  padding: 10px;
  margin-bottom: 10px;
}
.message {
  margin: 5px 0;
}
.input-box {
  display: flex;
  gap: 10px;
}
input {
  flex: 1;
  padding: 5px;
}
button {
  padding: 5px 10px;
}
</style>

```
### Vue.js
**reactivity**
1. _Reactive Data_: In Vue, data properties are made reactive, meaning changes to these properties automatically trigger updates to the DOM.
2. _Data Observation_: Vue observes changes to your data properties using a reactivity system. This is typically achieved through the use of getters and setters.
3. _Dependency Tracking_: When a reactive property is accessed, Vue tracks the dependencies (components or other parts of the application that use this property).
4. _Automatic Updates_: When a reactive property is mutated, Vue automatically triggers updates to all components or DOM elements that depend on this property.
  
**data ﬂow between components**  
Data flow between components is possible by using state management  
  
**most common cause of memory leaks**
1. Unreleased Event Listeners
   - Cause: Event listeners that are not removed when components are destroyed.
   - Solution: Use the beforeDestroy lifecycle hook to remove event listeners using this.$off(eventName, handler).

2. Global Event Bus Leakage
   - Cause: Components not being removed from the global event bus when they are destroyed.

   - Solution: Remove components from the event bus in the beforeDestroy hook using EventBus.$off(eventName).

3. Unused References
   - Cause: Holding references to DOM elements or Vue instances that are no longer needed.

   - Solution: Set references to null or undefined in the beforeDestroy hook to allow garbage collection.

4. Improper Cleanup of Third-Party Libraries
   - Cause: Not properly cleaning up resources from third-party libraries.

   - Solution: Follow the library's documentation to properly clean up resources in the beforeDestroy hook.

5. Memory Leaks from Plugins
   - Cause: Plugins that create memory leaks by not properly handling cleanup.

   - Solution: Ensure plugins are properly initialized and cleaned up in the component lifecycle hooks.

6. Memory Leaks from Persistent Storage
   - Cause: Persistent storage (like IndexedDB or LocalStorage) not being cleared properly.

   - Solution: Clear storage when it is no longer needed, especially before the component is destroyed.

7. Memory Leaks from Closures
   - Cause: Closures capturing large objects or DOM elements.

   - Solution: Be cautious with closures and avoid capturing large objects or DOM elements unless necessary.

8. Memory Leaks from Observers
   - Cause: Observers that are not removed when no longer needed.

   - Solution: Remove observers in the beforeDestroy hook.

9. Memory Leaks from Asynchronous Operations
   - Cause: Asynchronous operations that hold references to components or data.

   - Solution: Ensure that references are cleared and operations are properly handled in the beforeDestroy hook.

10. Memory Leaks from Data Binding
   - Cause: Improper data binding that leads to retained references.

   - Solution: Ensure data binding is properly managed and references are cleared when components are destroyed.
  
**state management**  
Simple State Management with Reactivity API  

**diﬀerence between pre-rendering and server side rendering**  
_Pre-rendering_  
Pre-rendering generates static HTML files at build time for each page of your application. This approach is also known as static site generation (SSG).  
During the build process, each page is rendered to HTML. The generated HTML files are served to the client.  
Advantages:  

- Performance: Since the HTML is pre-generated, pages load very quickly.

- SEO: Pre-rendered pages are fully rendered HTML, making them easily crawlable by search engines.

- Hosting: Can be hosted on any static hosting service without the need for a server.

- Use Cases: Content-heavy sites like blogs or documentation where content doesn't change frequently.

_Server-Side Rendering (SSR)_  
SSR renders your Vue.jsapplication to HTML on the server for each request. This is typically done using a server (like Node.js).
On each request, the server generates the HTML for the page and sends it to the client. The client receives a fully-rendered HTML page.  
Advantages:  

- Dynamic Content: Can handle dynamic content and user-specific data.

- SEO: SSR provides fully-rendered HTML pages, making them SEO-friendly.

- Initial Load Speed: Faster initial page load since the content is rendered on the server.

- Use Cases: Applications that require dynamic, user-specific content. E-commerce sites, dashboards, and applications with frequently changing data.
 
### Website Security Best Practises
1. Use HTTPS
2. Regularly Update Software
3. Implement Strong Authentication
4. Input Validation
5. Use Web Application Firewalls (WAF)
6. Secure Server Configuration
7. Backup Regularly
8. Monitor and Log Activity
9. Use Security Headers
10. Restrict File Uploads
11. Limit User Permissions
12. Implement Content Security Policy (CSP)
13. Session Management
14. Conduct Regular Security Audits
15. Educate and Train Users
16. Limit Access by IP
17. Use Strong Encryption
18. Implement Rate Limiting
19. Use Secure Development Practices
20. Monitor Third-Party Services

### Website Performance Best Practises
1. Optimize Images
2. Minimize HTTP Requests
3. Use a Content Delivery Network (CDN)
4. Minify CSS, JavaScript, and HTML
5. Enable Browser Caching
6. Optimize Web Fonts
7. Reduce Server Response Time
8. Limit Redirects
9. Use Asynchronous Loading for CSS and JavaScript
10. Choose a Reliable Web Hosting Provider
11. Monitor Performance Regularly
12. Implement Lazy Loading
13. Optimize Mobile Experience
14. Limit Third-Party Scripts
15. Use Efficient Code

### Golang
```
package main

import (
	"fmt"
	"regexp"
	"strings"
)

func main() {
	inStr := "Four, One two two three Three three four four four"
	str := strings.ToLower(inStr)
	re := regexp.MustCompile("[^a-zA-Z0-9]+")
	words := re.Split(str, -1)
	wordCount := make(map[string]int)

	for _, word := range words {
		if word != "" {
			count, exists := wordCount[word]
			if exists {
				wordCount[word] = count + 1
			} else {
				wordCount[word] = 1
			}
		}
	}

	for key, value := range wordCount {
		fmt.Printf("%s => %d\n", key, value)
	}
}
```

### Tools (1-5)
- Git = 5
- Redis = 2
- VSCode / JetBrains = 5
- Linux = 2
- AWS = 1
  - EC2
  - Lambda
  - RDS
  - Cloudwatch
  - S3
- Unit testing = 4
- Kanban boards = 5
