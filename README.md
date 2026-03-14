# telegram-
在tg中如果收藏夹中收藏的文件来源群组炸了收藏文件也会失效，这个项目旨在避免此事发生
import os
import asyncio
from telethon import TelegramClient, events

# --- 配置 ---
API_ID = 你的ID
API_HASH = '你的HASH'
DOWNLOAD_PATH = r'D:\TG_Cache'

if not os.path.exists(DOWNLOAD_PATH):
    os.makedirs(DOWNLOAD_PATH)

client = TelegramClient('session_tun_final', API_ID, API_HASH)

# 进度回调函数
def progress_callback(current, total, operation):
    percentage = 100 * current / total
    print(f"📊 {operation}: {percentage:.1f}% ({current // 1024 // 1024}MB / {total // 1024 // 1024}MB)", end='\r')

async def process_message(message):
    if message.media:
        is_vid = message.video or (message.document and message.document.mime_type.startswith('video/'))
        if is_vid:
            if message.text and "✅ 备份成功" in message.text:
                return
                
            file_name = f"video_{message.id}.mp4"
            print(f"\n🎬 开始处理视频 ID: {message.id}")
            
            try:
                # 1. 下载进度
                path = await message.download_media(
                    file=DOWNLOAD_PATH,
                    progress_callback=lambda c, t: progress_callback(c, t, "下载中")
                )
                print(f"\n✅ 下载完成，准备回传...")

                # 2. 上传进度
                await client.send_file(
                    'me', path, 
                    caption="✅ 备份成功",
                    progress_callback=lambda c, t: progress_callback(c, t, "上传中")
                )
                
                if os.path.exists(path):
                    os.remove(path)
                print(f"\n✨ 处理成功并已清理缓存")
            except Exception as e:
                print(f"\n❌ 出错: {e}")

@client.on(events.NewMessage(chats='me'))
async def handler(event):
    await process_message(event.message)

async def main():
    await client.start()
    print("🎉 登录成功！")
    
    choice = input("是否同步历史视频？(y/n): ").lower()
    if choice == 'y':
        async for message in client.iter_messages('me'):
            await process_message(message)

    print("\n📡 监控中...")
    await client.run_until_disconnected()

if __name__ == '__main__':
    asyncio.run(main())
