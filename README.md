# List App

A simple CRUD action To-Do List app with Rails API, React, and Heroku. The server-side is handled by Rails (Rails 5 introduced the [API-only mode](https://guides.rubyonrails.org/api_app.html)), which talks in JSON. The client-side is done in React, a popular frontend JavaScript framework.

Credit: [ReactJS + Ruby on Rails API + Heroku App Tutorial](https://medium.com/@bruno_boehm/reactjs-ruby-on-rails-api-heroku-app-2645c93f0814)

## Documentation
Issues I ran into following the tutorial and how to solve them: 

### 1. Step 2. Create the React Client
Received an error after setting the proxy as described in `package.json` with 
```json
  "name": "client,
  [...]
  "proxy": {
    "/api": {
      "target": "http://localhost:3001"
    }
  },
  [...[
```

```
When specified, "proxy" in package.json must be a string.
Instead, the type of "proxy" was "object".
Either remove "proxy" from package.json, or make it a string.
```
It is because you are using the advanced proxy configuration in which our `proxy` is an object.

Install the [http-proxy-middleware](https://github.com/facebook/create-react-app/issues/5103) package.(Read more: [Move advanced proxy configuration to src/setupProxy.js](https://github.com/facebook/create-react-app/issues/5103)). 

```zsh
$ yarn add http-proxy-middleware
```

Create `src/setupProxy.js` and place the `proxy` object as entries in the file
```js
const proxy = require('http-proxy-middleware')
 
module.exports = function(app) {
  app.use(proxy('/api', { target: 'http://localhost:3001/' }))
}
```
You goooood to API calls to the right port~

### 2. Step 7. Fetching API Data with Axios
You are running into a [CORS (Cross-Origin Resource Sharing)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) error. Your Rails API is not allowed by Access-Control-Allow-Origin.

Install gem [rack-cors](https://github.com/cyu/rack-cors)
```zsh
$ bundle
```
Navigate to `config/initializers/cors.rb` and uncomment these codes, and set origins to `localhost:3000`
```ruby
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'
    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```
No more CORS issues! 

### 3. Step 9-c. CR[U]D: UPDATE action
You seem to be creating an different list with the same id upon editing.
```
Warning: Encountered two children with the same key, `3`. Keys should be unique so that components maintain their identity across updates. Non-unique keys may cause children to be duplicated and/or omitted — the behavior is unsupported and could change in a future version.
```
Thank you [Medium user CMDS for posting the solution in the response section](https://medium.com/@brewNcode/within-the-success-response-in-editlist-id-title-excerpt-5e43bc31fc69).

You have to find the current list id that you are rending.
instead of 
```
const lists = this.state.lists;
lists[id-1] = {id, title, excerpt}
this.setState(() => ({
  ...
```
Change to: 
```
const index = lists.findIndex(list => list.id === id);
lists[index] = {id, title, excerpt}
this.setState(() => ({ ...
```

## Conclusion
I hope you find this documentation helpful!
