import telebot
from telebot import types
import sqlite3

bot = telebot.TeleBot('5923335843:AAHaKbkvCl_iLd5fpbCFVB2EYn__VwdhsCc')

# Инициализация базы данных
conn = sqlite3.connect('database.db')
cursor = conn.cursor()

# Удаление таблицы registration
cursor.execute("DROP TABLE IF EXISTS registration")

# Создание таблицы registration
cursor.execute("CREATE TABLE IF NOT EXISTS registration "
               "(chat_id INTEGER PRIMARY KEY, "
               "fio TEXT, "
               "prof_bilet TEXT, "
               "group_number TEXT)")

conn.commit()


cursor.execute('''
    CREATE TABLE IF NOT EXISTS events (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        event_name TEXT,
        event_date TEXT,
        event_description TEXT,
        photo_file_id JPG
    )
''')

cursor.execute("CREATE TABLE IF NOT EXISTS registration "
               "(chat_id INTEGER PRIMARY KEY, "
               "fio TEXT, "
               "prof_bilet TEXT, "
               "group_number TEXT)")

cursor.execute("CREATE TABLE IF NOT EXISTS events_registration "
               "(event_id INTEGER, "
               "chat_id INTEGER, "
               "PRIMARY KEY (event_id, chat_id), "
               "FOREIGN KEY (event_id) REFERENCES events(event_id), "
               "FOREIGN KEY (chat_id) REFERENCES registration(chat_id))")

conn.commit()
conn.close()


@bot.message_handler(commands=['start'])
def welcome(message):
    chat_id = message.chat.id

    # Отправка приветственного сообщения с фотографией
    photo = open('1.png', 'rb')
    bot.send_photo(chat_id, photo, caption='Добро пожаловать!\nЭто чат бот от Профбюро ВШПИМ ❤️💙\n\nЗдесь вы можете узнать подробную информацию о правовых тонкостях и о мероприятиях Профоюзой организации\n\nP.S.Если вы неккоректно ввели данные, пройдите регистрацию еще раз')

    # Создание клавиатуры с кнопками
    keyboard = types.InlineKeyboardMarkup(row_width=2)
    registration_button = types.InlineKeyboardButton(text='Регистрация', callback_data='registration')
    useful_info_button = types.InlineKeyboardButton(text='Полезная информация', callback_data='useful_info')
    show_events_button = types.InlineKeyboardButton(text='Показать мероприятия', callback_data='show_events')
    keyboard.add(registration_button, useful_info_button, show_events_button)

    bot.send_message(chat_id, 'Выберите одну из команд:', reply_markup=keyboard)


# Обработка нажатий на кнопки
@bot.callback_query_handler(func=lambda call: True)
def handle_button_click(call):
    chat_id = call.message.chat.id
    command = call.data

    if command == 'registration':
        bot.send_message(chat_id, 'Введите ваше полное имя:')
        bot.register_next_step_handler(call.message, process_registration_step)
    elif command == 'useful_info':
        keyboard = types.InlineKeyboardMarkup(row_width=1)
        button1 = types.InlineKeyboardButton(text='О дотациях', callback_data='dotations')
        button2 = types.InlineKeyboardButton(text='О материальной помощи', callback_data='financial_aid')
        button3 = types.InlineKeyboardButton(text='Вступить в профсоюз', callback_data='join_union')
        button4 = types.InlineKeyboardButton(text='Программы лояльности', callback_data='loyalty_programs')
        button5 = types.InlineKeyboardButton(text='О предоставлении академического отпуска',
                                             callback_data='academic_leave')
        button6 = types.InlineKeyboardButton(text='О стипендиях в Московском университете',
                                             callback_data='scholarships')

        keyboard.add(button1, button2, button3, button4, button5, button6)

        bot.send_message(chat_id, 'Выберите интересующую вас тему:', reply_markup=keyboard)
    elif command == 'dotations':
        bot.send_message(chat_id, 'Дотация это - выплата остронуждающимся студентам очной формы обучения.\n\nСумма выплаты дотации составляет 1200 рублей в месяц и выплачивается раз в квартал, то есть раз в 3 месяца.\nКатегории обучающихся, имеющих право на получение материальной поддержки: • Пострадавшие в результате Чернобыльской АЭС и других радиационных катастроф\n• Признанные в установленном порядке инвалидами 1, 2 группы и инвалидами с детства \n• Дети-инвалиды \n• Инвалиды и ветераны боевых действий \n• Обучающиеся, признанные в установленном порядке инвалидами 3 группы \n• Дети-сироты и дети, оставшиеся без попечения родителей \n• Обучающиеся из малоимущих/многодетных/неполных семей \n• Обучающиеся, потерявшие в период обучения одного или нескольких родителей \n• Студенты, страдающие хроническими заболеваниями\n\nПодать заявление на дотацию можно в личном кабинете Московского политеха в разделе справки и заявления')
    elif command == 'financial_aid':
        bot.send_message(chat_id, 'Материальная помощь назначается обучающимся очной формы обучения на бюджетной основе по личному заявлению.\n\nМатериальная помощь выплачивается на стипендиальную карту студента один раз в семестр. Сумма выплаты зависит от причины.\n\nКатегории обучающихся, имеющих право на получение материальной помощи: \n• Пострадавшие в результате Чернобыльской АЭС и других радиационных катастроф \n• Признанные в установленном порядке инвалидами 1, 2 группы и инвалидами с детства \n• Дети-инвалиды \n• Инвалиды и ветераны боевых-действий \n• Обучающиеся, признанные в установленном порядке инвалидами 3 группы \n• Дети-сироты и дети, оставшиеся без попечения родителей \n• Обучающиеся из малоимущих/многодетных/неполных семей \n• Обучающиеся, потерявшие в период обучения одного или нескольких родителей \n\nТакже материальную помощь можно получить по следующим причинам: \n• Компенсация расходов на проезд к месту проживания \n• Компенсация расходов в связи с мотивированной необходимостью улучшения жилищных условий \n• Временно оказавшиеся в сложной жизненной ситуации \n• Бракосочетание \n• Обучающиеся женского пола в связи с беременностью или рождением ребёнка \n• Пострадавшие при чрезвычайных обстоятельствах \n• В связи со смертью близкого родственника \n• Нуждающиеся в дорогостоящем лечении')
    elif command == 'join_union':
        bot.send_message(chat_id, 'Вступить в профсоз можнно онлайн по ссылке https://lk.eseur.ru/signup\nИли же лично подав заявление в кабинете В202 на большой Семёновской')
    elif command == 'loyalty_programs':
        bot.send_message(chat_id, 'Приложение «СКС РФ» позволяет профсоюзу всегда быть рядом. Одно из преимуществ приложения – дисконтная программа скидок и акций специально для студентов во всех городах-участниках по стране. \n\n Бонусная программа PROFCARDS - дисконтная программа скидок и акций, а также кэшбэка. Через партнеров PROFCARDS можно возвращать кэшбэк с покупок, пользоваться скидками и получать подарки. Также бонусы можно вывести на электронный кошелек, банковский счет или банковскую карту, а также пополнить баланс телефона.')
    elif command == 'academic_leave':
        bot.send_message(chat_id, 'Причины подачи заявления на академический отпуск: \n• Медицинские показания \n• Семейные обстоятельства \n• Призыв на военную службу \n• Иные обстоятельства \n\nАкадемический отпуск предоставляется неограниченное количество раз на срок не более двух лет. \nОбучающийся, находящийся в академическом отпуске не лишается получения материальной помощи, социальной стипендии, пользования социальной картой студента, летнего и зимнего оздоровления.')

    elif command == 'scholarships':
        bot.send_message(chat_id, 'В первом семестре назначается всем студентам-бюджетникам. После - тем, кто сдал экзаменационную сессию на «хорошо» и «отлично», без незачетов и пересдач. Для получения ГАС не надо подавать никаких заявлений, она автоматически начисляется по результатам сессии.\n\nРазмеры:\nДля всех студентов-бюджетников на 1-ом семестре 1-го курса - 2000 рублей\n\nПо результатам сессии:\nЕсли оценок “хорошо” больше, чем оценок “отлично” - 3000 рублей\nЕсли оценок “отлично” больше, чем оценок “хорошо” - 4500 рублей\nЕсли все оценки “отлично”, а также нет задолженностей - 6000 рублей (Количество промежуточных аттестаций: 2-3 - 7500 рублей; 4-5 - 10500 рублей; 6 и более - 13500 рублей)\nГАС выплачивается раз в месяц 25 числа, если дата не выпадает на выходные. Размер ГАС обновляется со следующего месяца после официальной даты окончания сессии по ее результатам.\nПГАС\nНазначается студентам за достижения в следующих видах деятельности:\n• культурно-творческой\n• общественной\n• учебной\n• спортивной\n• научно-исследовательской\n\nРазмеры:\nЗа достижения в культурно-творческой, общественной, спортивной деятельностях для всех курсов - 14000 рублей\nЗа достижения в научно-исследовательской и учебной деятельностях для 2 курса специалитета, бакалавриата, магистратуры - 15500 рублей\nЗа достижения в научно-исследовательской и учебной деятельностях для 3-4 курса специалитета и бакалавриата - 18000 рублей\nЗа достижения в научно-исследовательской и учебной деятельностях для 5-6 курса специалитета - 21000 рублей\nВ начале каждого семестра издается приказ ректора, в котором указываются сроки и условия приёма документов.\nПГАС\n\nГСС и ПГСС\nНазначается студентам - гражданам РФ бюджетной основы и очной формы обучения при наличии необходимых причин для ее получения. \n\nОбязательные документы (вне зависимости от причины): копия паспорта (стр. 2-5), копия регистрации в общежитии. \n\nРазмеры: \nРазмер ГСС - 3000 рублей \n\nСтуденты 1-го и 2-го курсов, назначенные на ГАС по итогам сессии, назначаются на повышенную социальную государственную стипендию (ПГСС) \nРазмер ПГСС - 12000 рублей')
    elif command == 'show_events':
        send_event_info(chat_id)






@bot.callback_query_handler(func=lambda call: call.data == 'show_reg')
def show_registered_users(call):
    chat_id = call.message.chat.id

    # Подключение к базе данных
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()

    # Получение всех зарегистрированных пользователей
    cursor.execute("SELECT * FROM registration")
    rows = cursor.fetchall()

    if rows:
        message = "Список зарегистрированных пользователей:\n\n"
        for row in rows:
            message += f"Chat ID: {row[0]}\n"
            message += f"ФИО: {row[1]}\n"
            message += f"Проф. билет: {row[2]}\n"
            message += f"Номер группы: {row[3]}\n\n"
    else:
        message = "Нет зарегистрированных пользователей."

    bot.send_message(chat_id, message)

    conn.close()



def process_registration_step(message):
    chat_id = message.chat.id
    fio = message.text
    bot.send_message(chat_id, 'Введите ваш номер профсоюзного билета билет:')
    bot.register_next_step_handler(message, process_prof_bilet_step, fio)


def process_prof_bilet_step(message, fio):
    chat_id = message.chat.id
    prof_bilet = message.text
    bot.send_message(chat_id, 'Введите номер вашей группы:')
    bot.register_next_step_handler(message, process_group_number_step, fio, prof_bilet)


def process_group_number_step(message, fio, prof_bilet):
    chat_id = message.chat.id
    group_number = message.text
    save_registration(chat_id, fio, prof_bilet, group_number)
    bot.send_message(chat_id, 'Регистрация успешно завершена!')


# Функция для обновления регистрации пользователя
def update_registration(message):
    chat_id = message.chat.id
    fio = message.text
    bot.send_message(chat_id, 'Введите ваш новый студенческий билет:')
    bot.register_next_step_handler(message, update_prof_bilet_step, fio)

def update_prof_bilet_step(message, fio):
    chat_id = message.chat.id
    prof_bilet = message.text
    bot.send_message(chat_id, 'Введите ваш новый номер группы:')
    bot.register_next_step_handler(message, update_group_number_step, fio, prof_bilet)

def update_group_number_step(message, fio, prof_bilet):
    chat_id = message.chat.id
    group_number = message.text
    db.update_registration(chat_id, fio=fio, prof_bilet=prof_bilet, group_number=group_number)
    bot.send_message(chat_id, 'Данные успешно обновлены!')
def save_registration(chat_id, fio, prof_bilet, group_number):
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    cursor.execute("INSERT INTO registration (chat_id, fio, prof_bilet, group_number) VALUES (?, ?, ?, ?)",
                   (chat_id, fio, prof_bilet, group_number))
    conn.commit()
    conn.close()


def send_event_info(chat_id):
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM events")
    events = cursor.fetchall()

    if len(events) == 0:
        bot.send_message(chat_id, 'В данный момент нет доступных мероприятий.')
    else:
        for event in events:
            event_id, event_name, event_date, event_description, photo_file_id = event
            message = f"Название: {event_name}\nДата: {event_date}\nОписание: {event_description}"
            if photo_file_id:
                bot.send_photo(chat_id, photo_file_id, caption=message)
            else:
                bot.send_message(chat_id, message)

    conn.close()


@bot.message_handler(commands=['add_event'])
def add_event(message):
    chat_id = message.chat.id
    bot.send_message(chat_id, 'Введите название мероприятия:')
    bot.register_next_step_handler(message, process_event_name_step)


def process_event_name_step(message):
    chat_id = message.chat.id
    event_name = message.text
    bot.send_message(chat_id, 'Введите дату мероприятия (в формате ДД.ММ.ГГГГ):')
    bot.register_next_step_handler(message, process_event_date_step, event_name)


def process_event_date_step(message, event_name):
    chat_id = message.chat.id
    event_date = message.text
    bot.send_message(chat_id, 'Введите описание мероприятия:')
    bot.register_next_step_handler(message, process_event_description_step, event_name, event_date)


def process_event_description_step(message, event_name, event_date):
    chat_id = message.chat.id
    event_description = message.text

    # Получение фотографии мероприятия от пользователя
    bot.send_message(chat_id, 'Пришлите фотографию мероприятия:')
    bot.register_next_step_handler(message, process_event_photo_step, event_name, event_date, event_description)


def create_table():
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS events
                      (id INTEGER PRIMARY KEY AUTOINCREMENT,
                       event_name TEXT,
                       event_date TEXT,
                       event_description TEXT,
                       photo_file_id TEXT)''')
    conn.commit()
    conn.close()


def process_event_photo_step(message, event_name, event_date, event_description):
    chat_id = message.chat.id

    # Проверяем, прислал ли пользователь фотографию
    if message.photo:
        # Получаем информацию о фотографии
        photo = message.photo[-1]  # Берем последнюю (самую большую) фотографию из присланных
        photo_file_id = photo.file_id

        # Сохраняем информацию о мероприятии в базу данных
        save_event(chat_id, event_name, event_description, photo_file_id)
        bot.send_message(chat_id, 'Мероприятие успешно добавлено!')
    else:
        # Сохраняем информацию о мероприятии без фотографии
        save_event(chat_id, event_name, event_description, None)
        bot.send_message(chat_id, 'Мероприятие успешно добавлено (без фотографии)!')


def save_event(chat_id, event_name, event_description, photo_file_id):
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()

    cursor.execute("INSERT INTO events (event_name, event_date, event_description, photo_file_id) "
                   "VALUES (?, NULL, ?, ?)", (event_name, event_description, photo_file_id))
    conn.commit()
    conn.close()


@bot.message_handler(commands=['delete_event'])
def delete_event(message):
    chat_id = message.chat.id
    bot.send_message(chat_id, 'Введите название мероприятия, которое нужно удалить:')
    bot.register_next_step_handler(message, process_delete_event_step)


def process_delete_event_step(message):
    chat_id = message.chat.id
    event_name = message.text
    delete_event_by_name(chat_id, event_name)


def delete_event_by_name(chat_id, event_name):
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    cursor.execute("DELETE FROM events WHERE event_name = ?", (event_name,))
    conn.commit()
    conn.close()
    bot.send_message(chat_id, 'Мероприятие успешно удалено!')


@bot.message_handler(commands=['show_events'])
def show_events(message):
    chat_id = message.chat.id

    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()

    cursor.execute("SELECT * FROM events")

    events = cursor.fetchall()

    if events:
        for event in events:
            event_name = event[1]
            event_date = event[2]
            event_description = event[3]
            photo_file_id = event[4]

            # Отправляем сообщение с информацией о мероприятии и фотографией
            message = f"Название: {event_name}\nДата: {event_date}\nОписание: {event_description}"
            if photo_file_id:
                bot.send_photo(chat_id, photo_file_id, caption=message)
            else:
                bot.send_message(chat_id, message)
    else:
        bot.send_message(chat_id, "Нет доступных мероприятий.")

    conn.close()

@bot.message_handler(commands=['show_reg'])
def show_registrations(message):
    chat_id = message.chat.id

    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()

    cursor.execute("SELECT * FROM registration")

    registrations = cursor.fetchall()

    if registrations:
        for reg in registrations:
            chat_id = reg[0]
            fio = reg[1]
            prof_bilet = reg[2]
            group_number = reg[3]

            message = f"Chat ID: {chat_id}\nФИО: {fio}\nПрофбилет: {prof_bilet}\nНомер группы: {group_number}"
            bot.send_message(chat_id, message)
    else:
        bot.send_message(chat_id, "Нет зарегистрированных пользователей.")

    conn.close()


bot.polling()
