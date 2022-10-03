# -
Всем привет, я вообще не разбираюсь в кодах и на просторах интернета набрел на граббер, взял скрипт, подкорректировал под себя, запустил, вылезает ошибка “TypeError: ‘function’ object is not iterable”
Подскажите пожалуйста что необходимо написать, чтобы ее исправить

from pyrogram import Client, filters
import shelve 
db = shelve.open('data.db', writeback=True)
 

API_ID = 189427401492
API_HASH = '87d63b57bf50a00681a11f194920592952'
PRIVATE_PUBLIC = '@'  # скрытый паблик для управления ботом
PUBLIC_PUBLIC = '@prizyvspbpiter'  # паблик куда будем репостить
SOURCE_PUBLICS = [
    # список пабликов-доноров, откуда бот будет пересылать посты
    '@hereyoucandoalot',
    '@u_now',
    '@rian_ru',
    '@ostorozhno_novosti'
]
PHONE_NUMBER = '+79811177235'  # номер зарегистрованный в телеге

app = Client("cyberpunk", api_id=API_ID, api_hash=API_HASH,
             phone_number=PHONE_NUMBER)
@app.on_message(filters.chat(SOURCE_PUBLICS))
def new_channel_post(client, message):
    post_id = add_post_to_db(message)
    
    message.forward(PRIVATE_PUBLIC)
    client.send_message(PRIVATE_PUBLIC, post_id)
    # потом для пересылки в публичный паблик админ должен отправить боту этот
def add_post_to_db(message):
    try:
     
        new_id = max(int(k) for k in db.keys()
                     if k.isdigit()) + 1
    except:
        # если постов еще нет в базе вылетит ошибка и мы попадем сюда
        # тогда id ставим = 1
        new_id = 1
    # запись в базу необходимой информации про пост
    # Обратите внимание, shelve поддеживает только строковые ключи
    db[str(new_id)] = {
        'username': message.chat.username,  # паблик-донор
        'message_id': message,  # внутренний id сообщения
    }
    return new_id

@app.on_message(filters.chat(PRIVATE_PUBLIC)
                & filters.regex(r'\d+\+')  # фильтр текста сообщения {число}+
                )
def post_request(client, message):
    # получаем id поста из сообщения (обрезаем "+" в конце)
    post_id = str(message.text).strip('+')
    # получаем из базы пост по этому id
    post = db.get(post_id)
    if post is None:
        # если нет в базе пишем в скрытый паблик ошибку
        client.send_message(PRIVATE_PUBLIC,
                            'ERROR NO POST ID IN DB')
        # и выходим
        return
    try:
        # по данным из базы, получаем pyrogram обьект сообщения
        msg = client.get_messages(post['username'], post['message_id'])
        # пересылаем его в паблик
        # as_copy=True значит, что мы не будем отображать паблик донор, будто это наш пост XD
        msg.forward(PUBLIC_PUBLIC, as_copy=True)
        # отправляем сообщение в скрытый паблик о успехе
        client.send_message(PRIVATE_PUBLIC, f'SUCCESS REPOST!')
    except Exception as e:
        # если произойдет какая-то ошибка в 3 строчках выше - сообщим админу
        client.send_message(PRIVATE_PUBLIC, f'ERROR {e}')
if name == '__main__':
    print('я в базе')
    app.run()  # эта строка запустит все обработчики
