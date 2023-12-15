import psycopg2

conn = psycopg2.connect(database="HW_psql_py", user="postgres", password="1")
def drop_table(conn):
    with conn.cursor() as cur:
        cur.execute("""
        DROP TABLE phonenumbers;
        DROP TABLE clients;
        """)
        conn.commit()
##1.Функция, создающая структуру БД (таблицы)##
def create_db(conn):
    with conn.cursor() as cur:
        cur.execute("""
        CREATE TABLE IF NOT EXISTS clients(
        id SERIAL PRIMARY KEY,
        name VARCHAR(100)NOT NULL,
        lastname VARCHAR(100)NOT NULL,
        email VARCHAR(100) NOT NULL
        );
        """)
        cur.execute("""
        CREATE TABLE IF NOT EXISTS phonenumbers(
        id_phonenumber SERIAL PRIMARY KEY,
        client_id INTEGER NOT NULL REFERENCES clients(id),
        phonenumber VARCHAR(20) UNIQUE);
        """)
        conn.commit()
#2.Функция, позволяющая добавить нового клиента.##
def add_client (conn, name=None, lastname=None, email=None):
    with conn.cursor() as cur:
        cur.execute("""
            INSERT INTO clients (name, lastname, email)
            VALUES (%s, %s, %s)
            RETURNING id, name, lastname, email;
            """, (name, lastname, email))
        print(cur.fetchone())
        conn.commit()
##3.Функция, позволяющая добавить телефон для существующего клиента.##
def add_phone (conn, client_id, phonenumber):
    with conn.cursor() as cur:
        cur.execute("""
            INSERT INTO phonenumbers (client_id, phonenumber)
            VALUES (%s, %s)
            RETURNING client_id, phonenumber;
            """, (client_id, phonenumber))
        print(cur.fetchone())
        conn.commit()
##4.Функция, позволяющая изменить данные о клиенте.##
def edit_client(conn, id, name=None, lastname=None, email=None):
    if name is not None:
        with conn.cursor() as cur:
            cur.execute("""
            UPDATE clients
            SET name = %s WHERE id = %s
            """, (name, id))
            conn.commit()
    if lastname is not None:
        with conn.cursor() as cur:
            cur.execute("""
            UPDATE clients
            SET lastname = %s WHERE id = %s
            """, (lastname, id))
            conn.commit()
    if email is not None:
        with conn.cursor() as cur:
            cur.execute("""
            UPDATE clients
            SET email = %s WHERE id = %s
            """, (email, id))
            conn.commit()
    return 'Данные клиента изменены успешно'
##5.Функция, позволяющая удалить телефон для существующего клиента.##
def delete_client_number(conn, id):
    with conn.cursor() as cur:
        cur.execute("""
        DELETE FROM phonenumbers
        WHERE client_id = %s
        """, (id,))
        # cur.execute("""
        # DELETE FROM clients
        # WHERE id = %s
        # """, (id,))
        conn.commit()
##6.Функция, позволяющая удалить существующего клиента.##
def delete_client(conn, client_id):
    with conn.cursor() as cur:
        cur.execute("""
            DELETE FROM phonenumbers WHERE client_id=%s;
            """, (client_id,))
        cur.execute("""
            DELETE FROM clients WHERE id=%s;
            """, (client_id,))
        cur.execute("""
            SELECT * FROM clients;
            """)
        conn.commit()
        print(cur.fetchall())
##7.Функция, позволяющая найти клиента по его данным: имени, фамилии, email или телефону.##
def find_client (conn, name=None, lastname=None, email=None, phonenumber=None):
    if name is None:
        name = '%'
    else:
        name = '%' + name + '%'
    if lastname is None:
        surname = '%'
    else:
        surname = '%' + lastname + '%'
    if email is None:
        email = '%'
    else:
        email = '%' + email + '%'
    if phonenumber is None:
        with conn.cursor() as cur:
            cur.execute("""
                     SELECT clients.id, clients.name, clients.lastname, clients.email, clients.phonenumber FROM clients
                     JOIN phonenumbers ON clients.id = phonenumbers.client_id
                     WHERE clients.name LIKE %s AND clients.lastname LIKE %s
                     AND clients.email LIKE %s
                     """, (name, surname, email))
            print(cur.fetchall())
    else:
        with conn.cursor() as cur:
            cur.execute("""
                     SELECT clients.id, clients.name, clients.lastname, clients.email, phonenumbers.phonenumber FROM clients
                     LEFT JOIN phonenumbers ON clients.id = phonenumbers.client_id
                     WHERE clients.name LIKE %s AND clients.lastname LIKE %s
                     AND clients.email LIKE %s AND phonenumbers.phonenumber like %s
                     """, (name, surname, email, phonenumber))
            print(cur.fetchall())

# drop_table(conn)
# create_db(conn)
# add_client (conn, "Вася","Пупкин", "PupVS@mail.ru")
# add_client (conn, "Роман","Романов", "dfbdf@mail.ru")
# add_client (conn, "Руслан","Русланов", "l/k.@mail.ru")
# add_client (conn, "Максим","Максимов", "hnghn@mail.ru")
# add_phone (conn, 6,88002000600)
# add_phone (conn, 2,88002000700)
# add_phone (conn, 3,88002000800)
# add_phone (conn, 4,88002000900)
# edit_client(conn, 1, "Пуп", "Васькин", email=None)
# delete_client_number(conn, 6)
# delete_client(conn, 5)
# find_client (conn, None, None, None, "88002000900")

