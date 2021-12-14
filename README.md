# Game-of-life

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```
### How to Deploy to Heroku

1. change VUE_APP_API_URL & VUE_APP_WS_URL in .env.
2. this site is recommend using docker to deploy. 

```
heroku create
heroku container:push web
heroku container:release web
```

### Demo
[demo site](https://afternoon-fjord-92266.herokuapp.com/)

![Imgur](https://imgur.com/SwC0P2u.gif)