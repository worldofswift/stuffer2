# SES

## 1. > .env
```dotenv
MAIL_DRIVER=ses
SES_REGION=
#SES_KEY=
#SES_SECRET=
```

> Ключи не требуются при работе в среде через AMI-роли 

## 2. SES - Выходим из песочницы
По-умолчанию Вы не сможете отправлять письма ни на какие адреса кроме заявленных и подтвержденных
Для преодоления этой проблемы необходимо отправить письмо в AWS с просьбой о снятии запрета

### Примеры писем
#### 1. Для отправки самому себе в целях информирования о заказах

> 1. We just add our own emails address to be notified in certain cases (e.g. new order). And SNS is not suitable because we need to compose email from raw html.
> 2. We send emails only ourselves so no complaints or bounces are possible.
> 3. They just need to add/remove their emails from internal management system.
> 4. It is based on predictable hourly rate of orders but with big difference because it is hard to predict number of clients now before service launch.
> 5. The site is inactive at now due to the development process.
