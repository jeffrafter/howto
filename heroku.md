# Heroku

```
git remote add heroku HEROKU_REPO_URL
git push heroku master
heroku login
heroku pg:reset --app APP_NAME DB_NAME
heroku run --app APP_NAME rake db:migrate
heroku restart
heroku run rails console
heroku config:set `cat .env`

```