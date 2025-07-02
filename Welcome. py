import asyncio
import random
from telethon import TelegramClient, events
from telethon.tl.types import ChannelParticipantsAdmins, User, Message
import re

api_id = 22340540
api_hash = '264130c425cb6a107c99fa8c4155a078'

client = TelegramClient('session', api_id, api_hash)

# پیام‌های خوش‌آمدگویی
welcome_texts = ["خوش اومدی جانم", "خوش اومدی"]
# پیام‌هایی که نشانهٔ تشکرن (تحلیل ساده زبانی)
thank_you_keywords = ["مرسی", "ممنون", "thanks", "thank you", "tnx", "tnq", "❤️", "🙏", "😍"]

# کاربران تگ‌شده قبلی
welcomed_users = set()

@client.on(events.ChatAction)
async def welcome_new_user(event):
    if not event.user_joined and not event.user_added:
        return

    chat = await event.get_chat()
    me = await client.get_me()
    admins = await client.get_participants(chat, filter=ChannelParticipantsAdmins)
    admin_ids = {admin.id for admin in admins}

    # فقط اگر ربات ادمینه
    if me.id not in admin_ids:
        return

    user: User = await event.get_user()
    msg = random.choice(welcome_texts)

    if user.username:
        sent_msg = await event.reply(f"@{user.username} {msg}")
    else:
        sent_msg = await event.reply(f"[{user.first_name}](tg://user?id={user.id}) {msg}")

    # ذخیره کن که به این کاربر خوش‌آمد گفته شده
    welcomed_users.add((chat.id, user.id, sent_msg.id))

@client.on(events.NewMessage())
async def reply_to_thank_you(event: events.NewMessage.Event):
    if not event.is_reply or not event.message:
        return

    try:
        original_msg: Message = await event.get_reply_message()
    except:
        return

    user = event.sender
    chat_id = event.chat_id

    # فقط اگر این شخص قبلاً با پیام ربات تگ شده بوده
    was_welcomed = any(
        chat_id == cid and user.id == uid and original_msg.id == mid
        for (cid, uid, mid) in welcomed_users
    )

    if not was_welcomed:
        return

    # تشخیص وجود کلمات تشکر
    msg_text = event.raw_text.lower()
    if any(k in msg_text for k in thank_you_keywords):
        await event.reply("☺️ معرفی کن خودتو")
        # یک بار واکنش نشون بده، نه بیشتر
        welcomed_users.discard((chat_id, user.id, original_msg.id))

async def main():
    await client.start()
    print("[✓] ربات فعال شد. منتظر ورود اعضای جدید و پاسخ به تشکر هستم...")
    await client.run_until_disconnected()

client.loop.run_until_complete(main())
