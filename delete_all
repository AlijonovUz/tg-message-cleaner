import asyncio
import os
from telethon import TelegramClient, errors
from telethon.sessions import StringSession
from telethon.errors import SessionPasswordNeededError

API_ID = 12345678
API_HASH = 'cef....'
SESSION_FILE = "session.txt"
CHANNEL = "foniy_hayat"
batch_size = 100

client = None

async def create_or_load_session():
    global client
    if os.path.exists(SESSION_FILE):
        session_str = open(SESSION_FILE).read().strip()
        client = TelegramClient(StringSession(session_str), API_ID, API_HASH)
    else:
        client = TelegramClient(StringSession(), API_ID, API_HASH)
        await client.connect()
        if not await client.is_user_authorized():
            phone = input("Telefon raqamingiz (+998...): ")
            await client.send_code_request(phone)
            while True:
                code = input("SMS kod: ")
                try:
                    await client.sign_in(phone=phone, code=code)
                    break
                except SessionPasswordNeededError:
                    while True:
                        password = input("2FA parol: ")
                        try:
                            await client.sign_in(password=password)
                            break
                        except Exception:
                            print("Parol xato")
                    break
                except Exception:
                    print("Kod xato yoki noto‘g‘ri")
        session_str = client.session.save()
        open(SESSION_FILE, "w").write(session_str)
        try:
            os.chmod(SESSION_FILE, 0o600)
        except:
            pass

async def delete_batch(client, entity, ids):
    try:
        await client.delete_messages(entity, ids)
        print(f"O'chirildi: {len(ids)} ta xabar")
        await asyncio.sleep(1)
    except errors.FloodWaitError as e:
        print(f"FloodWaitError: {e.seconds}s kutish kerak")
        await asyncio.sleep(e.seconds)
        await delete_batch(client, entity, ids)

async def main():
    global client
    await create_or_load_session()
    await client.start()
    print("Boshlanmoqda, barcha xabarlar tozalanadi:", CHANNEL)
    to_delete_ids = []
    count_found = 0
    async for msg in client.iter_messages(CHANNEL, limit=None):
        count_found += 1
        to_delete_ids.append(msg.id)
        if len(to_delete_ids) >= batch_size:
            await delete_batch(client, CHANNEL, to_delete_ids)
            to_delete_ids.clear()
    if to_delete_ids:
        await delete_batch(client, CHANNEL, to_delete_ids)
    print(f"Tugatildi. Jami o'chirilgan xabarlar: {count_found}")
    await client.disconnect()

if __name__ == "__main__":
    asyncio.run(main())
