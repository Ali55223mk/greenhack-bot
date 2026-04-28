#!/usr/bin/env python3
"""
╔══════════════════════════════════════════════════╗
║            🟢  GreenHack Bot  🟢                 ║
║      بوت الأمن السيبراني والبرمجة الاحترافي      ║
║                   v3.0                           ║
╚══════════════════════════════════════════════════╝
"""

import logging
import socket
import ssl as ssl_lib
import hashlib
import base64
import codecs
import re
import ipaddress
import secrets
import string
import uuid
import subprocess
import sys
import platform
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import (
    Application, CommandHandler, MessageHandler,
    CallbackQueryHandler, ContextTypes, filters,
)

# ══════════════════════════════════════════════
#  ضع توكن البوت هنا  (من @BotFather)
# ══════════════════════════════════════════════
BOT_TOKEN = "8598787104:AAE5b4VDDrVIcTjWt7UwqxpuEyIis4vFP7E"

logging.basicConfig(
    format="%(asctime)s | %(levelname)s | %(message)s",
    level=logging.INFO,
    handlers=[logging.FileHandler("greenhack.log"), logging.StreamHandler()],
)
log = logging.getLogger(__name__)

# ══════════════════════════════════════════════
#  ثوابت مشتركة
# ══════════════════════════════════════════════
MORSE_CODE = {
    'A':'.-','B':'-...','C':'-.-.','D':'-..','E':'.','F':'..-.','G':'--.','H':'....','I':'..','J':'.---',
    'K':'-.-','L':'.-..','M':'--','N':'-.','O':'---','P':'.--.','Q':'--.-','R':'.-.','S':'...','T':'-',
    'U':'..-','V':'...-','W':'.--','X':'-..-','Y':'-.--','Z':'--..','0':'-----','1':'.----','2':'..---',
    '3':'...--','4':'....-','5':'.....','6':'-....','7':'--...','8':'---..','9':'----.', ' ':'/',
    '.':'.-.-.-',',':'--..--','?':'..--..','!':'-.-.--','-':'-....-','/':'-..-.','@':'.--.-.','(':'-.--.',')':'-.--.-',
}
MORSE_REVERSE = {v: k for k, v in MORSE_CODE.items()}

GLOSSARY_DB = {
    "xss":      "🔴 *Cross-Site Scripting (XSS)*\nحقن سكريبت خبيث في صفحة ويب يُنفَّذ في متصفح الضحية",
    "sqli":     "🔴 *SQL Injection*\nحقن أوامر SQL للتلاعب بقواعد البيانات",
    "csrf":     "🟠 *Cross-Site Request Forgery*\nإجبار المستخدم على تنفيذ طلبات غير مقصودة",
    "mitm":     "🔴 *Man-in-the-Middle*\nالتنصت وتعديل الاتصالات بين طرفين",
    "rce":      "🔴 *Remote Code Execution*\nتنفيذ كود عشوائي عن بُعد — خطورة قصوى",
    "lfi":      "🔴 *Local File Inclusion*\nقراءة ملفات محلية عبر ثغرة ويب",
    "ssrf":     "🔴 *Server-Side Request Forgery*\nإجبار الخادم على إرسال طلبات داخلية",
    "idor":     "🟡 *Insecure Direct Object Reference*\nالوصول لموارد مستخدمين آخرين بتغيير المعرّف",
    "dos":      "🟠 *Denial of Service*\nإغراق الخادم لإسقاطه",
    "ddos":     "🔴 *Distributed DoS*\nإغراق موزع من آلاف الأجهزة",
    "pentest":  "✅ *Penetration Testing*\nاختبار اختراق مصرح به لكشف الثغرات",
    "payload":  "💣 *Payload*\nالكود أو البيانات الخبيثة المُرسَلة في الهجوم",
    "firewall": "🛡️ *Firewall*\nجدار الحماية — يتحكم في حركة الشبكة",
    "vpn":      "🔒 *VPN*\nشبكة خاصة افتراضية لتشفير الاتصال",
    "zero-day": "⚡ *Zero-Day*\nثغرة غير معروفة ليس لها تصحيح بعد",
    "brute":    "🔓 *Brute Force*\nتجربة جميع الاحتمالات لكسر كلمة المرور",
    "phishing": "🎣 *Phishing*\nانتحال هوية جهة موثوقة لسرقة البيانات",
    "malware":  "🦠 *Malware*\nأي برنامج خبيث: فيروس، تروجان، رانسوم...",
    "ransomware":"💰 *Ransomware*\nبرنامج يشفر ملفاتك ويطلب فدية",
    "cve":      "📋 *CVE*\nقاعدة بيانات الثغرات الأمنية المعروفة",
}

# ══════════════════════════════════════════════
#  /start
# ══════════════════════════════════════════════
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user.first_name
    keyboard = [
        [InlineKeyboardButton("🔍 فحص الشبكة",   callback_data="m_net"),
         InlineKeyboardButton("🔐 التشفير",        callback_data="m_crypto")],
        [InlineKeyboardButton("🌐 فحص الويب",      callback_data="m_web"),
         InlineKeyboardButton("💻 أدوات المبرمج",  callback_data="m_code")],
        [InlineKeyboardButton("📚 تعليم سيبراني",  callback_data="m_learn"),
         InlineKeyboardButton("📖 قاموس",          callback_data="m_glossary")],
        [InlineKeyboardButton("❓ جميع الأوامر",   callback_data="m_help")],
    ]
    await update.message.reply_text(
        f"🟢 *GreenHack Bot*\n\n"
        f"أهلاً *{user}* 👋\n"
        f"بوتك الاحترافي في الأمن السيبراني والبرمجة\n\n"
        f"اختر قسماً أو اكتب `/help` للأوامر الكاملة\n\n"
        f"⚠️ _للأغراض التعليمية فقط_",
        parse_mode="Markdown",
        reply_markup=InlineKeyboardMarkup(keyboard),
    )

# ══════════════════════════════════════════════
#  /help
# ══════════════════════════════════════════════
async def help_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    txt = """
🟢 *GreenHack Bot — الأوامر الكاملة*

━━ 🔍 *الشبكة* ━━
`/ping <host>` — اختبار الاتصال
`/portscan <ip> [start] [end]` — مسح منافذ (شبكة محلية)
`/dns <domain>` — استعلام DNS
`/ssl <domain>` — فحص شهادة SSL
`/myip` — IP الخادم الحالي

━━ 🔐 *التشفير* ━━
`/hash <نص>` — MD5 / SHA1 / SHA256 / SHA512
`/b64enc <نص>` — ترميز Base64
`/b64dec <نص>` — فك ترميز Base64
`/rot13 <نص>` — تشفير ROT13
`/hex <نص>` — تحويل إلى Hex
`/unhex <hex>` — تحويل Hex إلى نص
`/morse <نص>` — تحويل إلى موشير
`/unmorse <رمز>` — فك تشفير موشير

━━ 🛠️ *أدوات* ━━
`/passgen [طول]` — توليد كلمة مرور قوية
`/uuid` — توليد UUID عشوائي
`/regex <نمط> <نص>` — اختبار RegEx
`/encode <نص>` — ترميزات متعددة دفعة واحدة

━━ 📚 *تعليم* ━━
`/owasp` — OWASP Top 10
`/roadmap` — خارطة طريق السيبراني
`/ctf` — مصادر CTF والمنصات
`/glossary [مصطلح]` — قاموس المصطلحات
`/ports` — جدول المنافذ المعروفة

━━ ℹ️ *أخرى* ━━
`/about` — عن البوت
`/start` — القائمة الرئيسية
"""
    await update.message.reply_text(txt, parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /ping
# ══════════════════════════════════════════════
async def ping_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/ping <ip أو domain>`", parse_mode="Markdown")

    target = context.args[0]
    if not re.match(r'^[a-zA-Z0-9.\-_]+$', target):
        return await update.message.reply_text("❌ مدخل غير صالح")

    await update.message.reply_text(f"📡 جاري فحص `{target}`...", parse_mode="Markdown")
    try:
        # تحديد أمر ping حسب نظام التشغيل
        flag = "-n" if platform.system().lower() == "windows" else "-c"
        result = subprocess.run(
            ["ping", flag, "4", target],
            capture_output=True, text=True, timeout=15
        )
        out = (result.stdout or result.stderr).strip()
        lines = out.split('\n')
        # نأخذ آخر 4 أسطر (الملخص)
        summary = '\n'.join(lines[-4:])
        status = "✅ متاح" if result.returncode == 0 else "❌ لا يستجيب"
        await update.message.reply_text(
            f"📡 *Ping → `{target}`*  {status}\n\n```\n{summary}\n```",
            parse_mode="Markdown"
        )
    except FileNotFoundError:
        # ping غير موجود — نستخدم socket بديلاً
        try:
            ip = socket.gethostbyname(target)
            s = socket.create_connection((target, 80), timeout=3)
            s.close()
            await update.message.reply_text(
                f"✅ `{target}` متاح  (`{ip}`)\n_(TCP port 80 open)_",
                parse_mode="Markdown"
            )
        except Exception:
            try:
                ip = socket.gethostbyname(target)
                await update.message.reply_text(
                    f"🟡 `{target}` → `{ip}` (DNS يعمل، لكن تعذّر الاتصال المباشر)",
                    parse_mode="Markdown"
                )
            except Exception:
                await update.message.reply_text(f"❌ تعذّر الوصول إلى `{target}`", parse_mode="Markdown")
    except subprocess.TimeoutExpired:
        await update.message.reply_text(f"⏰ انتهت المهلة — `{target}` لا يستجيب", parse_mode="Markdown")
    except Exception as e:
        await update.message.reply_text(f"❌ خطأ: `{e}`", parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /portscan  (شبكة محلية فقط)
# ══════════════════════════════════════════════
async def portscan_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text(
            "❌ الاستخدام: `/portscan <ip> [start] [end]`\n"
            "مثال: `/portscan 192.168.1.1 1 100`\n"
            "⚠️ يعمل على الشبكات المحلية فقط",
            parse_mode="Markdown"
        )

    target = context.args[0]
    try:
        ip = socket.gethostbyname(target)
    except socket.gaierror:
        return await update.message.reply_text("❌ تعذّر حل العنوان")

    try:
        ip_obj = ipaddress.ip_address(ip)
        if not (ip_obj.is_private or ip_obj.is_loopback):
            return await update.message.reply_text(
                "⚠️ *لأسباب أمنية وقانونية* يُسمح بمسح الشبكات المحلية فقط.\n\n"
                "الشبكات المسموحة:\n`192.168.x.x / 10.x.x.x / 172.16-31.x.x / 127.x.x.x`\n\n"
                "_استخدم أداة مثل nmap على بيئتك الخاصة_",
                parse_mode="Markdown"
            )
    except ValueError:
        return await update.message.reply_text("❌ عنوان IP غير صالح")

    try:
        start_p = int(context.args[1]) if len(context.args) > 1 else 1
        end_p   = int(context.args[2]) if len(context.args) > 2 else 1024
    except ValueError:
        return await update.message.reply_text("❌ المنافذ يجب أن تكون أرقاماً")

    start_p = max(1, min(start_p, 65535))
    end_p   = max(1, min(end_p,   65535))
    if start_p > end_p:
        start_p, end_p = end_p, start_p
    if end_p - start_p > 1000:
        return await update.message.reply_text("⚠️ الحد الأقصى 1000 منفذ لكل عملية مسح")

    await update.message.reply_text(
        f"🔍 مسح `{ip}` المنافذ {start_p}–{end_p}...",
        parse_mode="Markdown"
    )

    open_ports = []
    COMMON_SERVICES = {
        21:'FTP',22:'SSH',23:'Telnet',25:'SMTP',53:'DNS',80:'HTTP',
        110:'POP3',143:'IMAP',443:'HTTPS',445:'SMB',3306:'MySQL',
        3389:'RDP',5432:'PostgreSQL',6379:'Redis',8080:'HTTP-Alt',8443:'HTTPS-Alt',
    }

    for port in range(start_p, end_p + 1):
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.settimeout(0.3)
            if s.connect_ex((ip, port)) == 0:
                svc = COMMON_SERVICES.get(port, "?")
                open_ports.append(f"  ✅ {port:<6} {svc}")
            s.close()
        except Exception:
            pass

    if open_ports:
        body = "\n".join(open_ports)
        await update.message.reply_text(
            f"🔓 *{len(open_ports)} منفذ مفتوح على* `{ip}`\n```\n{body}\n```",
            parse_mode="Markdown"
        )
    else:
        await update.message.reply_text(
            f"🔒 لا توجد منافذ مفتوحة في النطاق {start_p}–{end_p} على `{ip}`",
            parse_mode="Markdown"
        )

# ══════════════════════════════════════════════
#  /dns
# ══════════════════════════════════════════════
async def dns_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/dns <domain>`", parse_mode="Markdown")

    domain = context.args[0].replace("https://","").replace("http://","").split("/")[0]
    try:
        ipv4 = socket.gethostbyname(domain)
        all_info = socket.getaddrinfo(domain, None)
        ips = list({a[4][0] for a in all_info})
        await update.message.reply_text(
            f"🌐 *DNS Lookup: `{domain}`*\n\n"
            f"```\nIPv4 الرئيسي : {ipv4}\n"
            f"كل العناوين : {', '.join(ips)}\n```",
            parse_mode="Markdown"
        )
    except socket.gaierror as e:
        await update.message.reply_text(f"❌ فشل DNS: `{e}`", parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /ssl
# ══════════════════════════════════════════════
async def ssl_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/ssl <domain>`", parse_mode="Markdown")

    domain = context.args[0].replace("https://","").replace("http://","").split("/")[0]
    await update.message.reply_text(f"🔒 جاري فحص SSL لـ `{domain}`...", parse_mode="Markdown")
    try:
        ctx = ssl_lib.create_default_context()
        conn = socket.create_connection((domain, 443), timeout=8)
        with ctx.wrap_socket(conn, server_hostname=domain) as s:
            cert = s.getpeercert()

        subject = dict(x[0] for x in cert.get("subject", []))
        issuer  = dict(x[0] for x in cert.get("issuer",  []))
        san     = cert.get("subjectAltName", [])
        san_str = ", ".join(v for _, v in san[:5])

        await update.message.reply_text(
            f"🔒 *SSL: `{domain}`*  ✅ صالحة\n\n"
            f"```\n"
            f"CN      : {subject.get('commonName','N/A')}\n"
            f"المُصدِر: {issuer.get('organizationName','N/A')}\n"
            f"تنتهي  : {cert.get('notAfter','N/A')}\n"
            f"SAN    : {san_str or 'N/A'}\n"
            f"```",
            parse_mode="Markdown"
        )
    except ssl_lib.SSLCertVerificationError as e:
        await update.message.reply_text(f"❌ شهادة غير موثوقة: `{e}`", parse_mode="Markdown")
    except ConnectionRefusedError:
        await update.message.reply_text(f"❌ المنفذ 443 مغلق على `{domain}`", parse_mode="Markdown")
    except socket.timeout:
        await update.message.reply_text(f"⏰ انتهت المهلة عند الاتصال بـ `{domain}`", parse_mode="Markdown")
    except Exception as e:
        await update.message.reply_text(f"❌ خطأ: `{e}`", parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /myip
# ══════════════════════════════════════════════
async def myip_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    try:
        import urllib.request
        ip = urllib.request.urlopen("https://api.ipify.org", timeout=6).read().decode().strip()
        await update.message.reply_text(f"🌍 *IP الخادم الحالي*\n\n`{ip}`", parse_mode="Markdown")
    except Exception as e:
        await update.message.reply_text(f"❌ تعذّر جلب IP: `{e}`", parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /hash
# ══════════════════════════════════════════════
async def hash_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/hash <نص>`", parse_mode="Markdown")

    text = " ".join(context.args).encode()
    await update.message.reply_text(
        f"🔐 *Hash: `{' '.join(context.args)}`*\n\n"
        f"```\n"
        f"MD5     : {hashlib.md5(text).hexdigest()}\n"
        f"SHA1    : {hashlib.sha1(text).hexdigest()}\n"
        f"SHA256  : {hashlib.sha256(text).hexdigest()}\n"
        f"SHA512  : {hashlib.sha512(text).hexdigest()}\n"
        f"SHA3-256: {hashlib.sha3_256(text).hexdigest()}\n"
        f"```",
        parse_mode="Markdown"
    )

# ══════════════════════════════════════════════
#  /b64enc & /b64dec
# ══════════════════════════════════════════════
async def b64enc_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/b64enc <نص>`", parse_mode="Markdown")
    text = " ".join(context.args)
    enc  = base64.b64encode(text.encode()).decode()
    await update.message.reply_text(
        f"🔒 *Base64 Encode*\n\n📥 `{text}`\n📤 `{enc}`",
        parse_mode="Markdown"
    )

async def b64dec_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/b64dec <نص مرمز>`", parse_mode="Markdown")
    try:
        raw = " ".join(context.args)
        dec = base64.b64decode(raw.encode()).decode()
        await update.message.reply_text(
            f"🔓 *Base64 Decode*\n\n📥 `{raw}`\n📤 `{dec}`",
            parse_mode="Markdown"
        )
    except Exception:
        await update.message.reply_text("❌ النص ليس Base64 صحيحاً")

# ══════════════════════════════════════════════
#  /rot13
# ══════════════════════════════════════════════
async def rot13_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/rot13 <نص>`", parse_mode="Markdown")
    text   = " ".join(context.args)
    result = codecs.encode(text, "rot_13")
    await update.message.reply_text(
        f"🔄 *ROT13*\n\n`{text}` ↔ `{result}`",
        parse_mode="Markdown"
    )

# ══════════════════════════════════════════════
#  /hex & /unhex
# ══════════════════════════════════════════════
async def hex_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/hex <نص>`", parse_mode="Markdown")
    text    = " ".join(context.args)
    raw_hex = text.encode().hex()
    spaced  = " ".join(raw_hex[i:i+2] for i in range(0, len(raw_hex), 2))
    await update.message.reply_text(
        f"🔢 *Text → Hex*\n\n📥 `{text}`\n📤 `{spaced}`",
        parse_mode="Markdown"
    )

async def unhex_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/unhex <hex>`", parse_mode="Markdown")
    try:
        raw = "".join(context.args).replace(" ","").replace("0x","")
        decoded = bytes.fromhex(raw).decode()
        await update.message.reply_text(
            f"🔓 *Hex → Text*\n\n📥 `{raw}`\n📤 `{decoded}`",
            parse_mode="Markdown"
        )
    except Exception:
        await update.message.reply_text("❌ قيمة Hex غير صالحة")

# ══════════════════════════════════════════════
#  /morse & /unmorse
# ══════════════════════════════════════════════
async def morse_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/morse <نص إنجليزي>`", parse_mode="Markdown")
    text   = " ".join(context.args).upper()
    result = " ".join(MORSE_CODE.get(c, "?") for c in text)
    await update.message.reply_text(
        f"📡 *Text → Morse*\n\n📥 `{text}`\n📤 `{result}`",
        parse_mode="Markdown"
    )

async def unmorse_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text(
            "❌ الاستخدام: `/unmorse <رمز موشير>`\nمثال: `/unmorse .... . .-.. .-.. ---`",
            parse_mode="Markdown"
        )
    code   = " ".join(context.args)
    result = "".join(MORSE_REVERSE.get(c, "?") for c in code.split(" "))
    await update.message.reply_text(
        f"📡 *Morse → Text*\n\n📥 `{code}`\n📤 `{result}`",
        parse_mode="Markdown"
    )

# ══════════════════════════════════════════════
#  /encode  — جميع الترميزات دفعة واحدة
# ══════════════════════════════════════════════
async def encode_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        return await update.message.reply_text("❌ الاستخدام: `/encode <نص>`", parse_mode="Markdown")
    text = " ".join(context.args)
    b    = text.encode()
    await update.message.reply_text(
        f"🔄 *ترميزات متعددة لـ:* `{text}`\n\n"
        f"```\n"
        f"Base64  : {base64.b64encode(b).decode()}\n"
        f"Hex     : {b.hex()}\n"
        f"ROT13   : {codecs.encode(text,'rot_13')}\n"
        f"MD5     : {hashlib.md5(b).hexdigest()}\n"
        f"SHA256  : {hashlib.sha256(b).hexdigest()}\n"
        f"```",
        parse_mode="Markdown"
    )

# ══════════════════════════════════════════════
#  /passgen
# ══════════════════════════════════════════════
async def passgen_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    try:
        length = int(context.args[0]) if context.args else 16
        length = max(8, min(length, 64))
    except ValueError:
        length = 16

    chars    = string.ascii_letters + string.digits + "!@#$%^&*()-_=+[]{}|;:<>?"
    passwords = ["".join(secrets.choice(chars) for _ in range(length)) for _ in range(5)]
    body      = "\n".join(f"{i+1}. {p}" for i, p in enumerate(passwords))
    await update.message.reply_text(
        f"🔑 *5 كلمات مرور قوية (طول {length})*\n\n```\n{body}\n```\n"
        f"⚠️ _لا تشاركها مع أحد_",
        parse_mode="Markdown"
    )

# ══════════════════════════════════════════════
#  /uuid
# ══════════════════════════════════════════════
async def uuid_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    uuids = "\n".join(f"{i+1}. {uuid.uuid4()}" for i in range(5))
    await update.message.reply_text(f"🆔 *5 UUIDs عشوائية*\n\n```\n{uuids}\n```", parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /regex
# ══════════════════════════════════════════════
async def regex_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if len(context.args) < 2:
        return await update.message.reply_text(
            "❌ الاستخدام: `/regex <pattern> <نص>`\n"
            "مثال: `/regex \\d+ hello123`",
            parse_mode="Markdown"
        )
    pattern = context.args[0]
    text    = " ".join(context.args[1:])
    try:
        matches = re.findall(pattern, text)
        if matches:
            await update.message.reply_text(
                f"✅ *{len(matches)} تطابق*\n\n"
                f"النمط: `{pattern}`\nالنص: `{text}`\n"
                f"النتائج: `{matches}`",
                parse_mode="Markdown"
            )
        else:
            await update.message.reply_text(f"❌ لا يوجد تطابق للنمط `{pattern}` في النص", parse_mode="Markdown")
    except re.error as e:
        await update.message.reply_text(f"❌ نمط RegEx خاطئ: `{e}`", parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /glossary
# ══════════════════════════════════════════════
async def glossary_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        keys = "  ".join(f"`{k}`" for k in GLOSSARY_DB)
        return await update.message.reply_text(
            f"📖 *قاموس مصطلحات الأمن السيبراني*\n\n"
            f"{keys}\n\n"
            f"الاستخدام: `/glossary <مصطلح>`\nمثال: `/glossary xss`",
            parse_mode="Markdown"
        )
    term = context.args[0].lower()
    if term in GLOSSARY_DB:
        await update.message.reply_text(GLOSSARY_DB[term], parse_mode="Markdown")
    else:
        await update.message.reply_text(
            f"❌ المصطلح `{term}` غير موجود\n"
            f"المتاح: {', '.join(GLOSSARY_DB.keys())}",
            parse_mode="Markdown"
        )

# ══════════════════════════════════════════════
#  /ports  — جدول المنافذ الشائعة
# ══════════════════════════════════════════════
async def ports_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    table = """
🌐 *المنافذ الشائعة*

```
المنفذ  البروتوكول  الوصف
─────────────────────────────
20/21   FTP         نقل الملفات
22      SSH         الاتصال الآمن
23      Telnet      اتصال غير آمن
25      SMTP        إرسال البريد
53      DNS         أسماء النطاقات
80      HTTP        الويب
110     POP3        استقبال البريد
143     IMAP        البريد (مزامنة)
443     HTTPS       الويب الآمن
445     SMB         مشاركة الملفات
3306    MySQL       قاعدة بيانات
3389    RDP         سطح المكتب البعيد
5432    PostgreSQL  قاعدة بيانات
6379    Redis       قاعدة بيانات
8080    HTTP-Alt    ويب بديل
8443    HTTPS-Alt   ويب آمن بديل
27017   MongoDB     قاعدة بيانات
```
"""
    await update.message.reply_text(table, parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /owasp
# ══════════════════════════════════════════════
async def owasp_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("""
🛡️ *OWASP Top 10 — 2021*

1️⃣ *Broken Access Control* — تجاوز صلاحيات الوصول
2️⃣ *Cryptographic Failures* — فشل التشفير وتسريب البيانات
3️⃣ *Injection* — SQL / XSS / Command Injection
4️⃣ *Insecure Design* — ثغرات في التصميم الأساسي
5️⃣ *Security Misconfiguration* — إعدادات أمنية خاطئة
6️⃣ *Vulnerable Components* — مكتبات قديمة وضعيفة
7️⃣ *Auth Failures* — ضعف المصادقة وإدارة الجلسات
8️⃣ *Software Integrity Failures* — عدم التحقق من التحديثات
9️⃣ *Logging Failures* — قصور في التسجيل والمراقبة
🔟 *SSRF* — Server-Side Request Forgery

🔗 _المصدر: owasp.org/Top10_
""", parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /roadmap
# ══════════════════════════════════════════════
async def roadmap_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("""
🗺️ *خارطة طريق الأمن السيبراني*

🟢 *المرحلة 1 — الأساسيات*
• شبكات: TCP/IP, OSI, DNS, HTTP/S
• Linux: أوامر أساسية، صلاحيات الملفات
• Python & Bash: أساسيات البرمجة
• المفاهيم: CIA Triad, Encryption, Hashing

🟡 *المرحلة 2 — الوسيط*
• اختبار الاختراق: Nmap, Metasploit, BurpSuite
• فحص الويب: OWASP Top 10, SQLMap
• الجنائيات: Autopsy, Volatility, Wireshark
• CTF: HackTheBox, TryHackMe, PicoCTF

🔴 *المرحلة 3 — المتقدم*
• Exploit Development & Buffer Overflow
• Malware Analysis & Reverse Engineering
• Cloud Security: AWS / Azure / GCP
• Red Team & Blue Team Operations
• Active Directory Attacks

🏆 *شهادات موصى بها*
`CompTIA Security+` → `CEH` → `OSCP` → `CISSP`
""", parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /ctf
# ══════════════════════════════════════════════
async def ctf_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("""
🏁 *مصادر CTF والتدريب*

🌐 *منصات التدريب*
• TryHackMe — مبتدئين (tryhackme.com)
• HackTheBox — متوسط/متقدم (hackthebox.com)
• PicoCTF — مجاني للمبتدئين (picoctf.org)
• PentesterLab — تخصص ويب
• VulnHub — آلات افتراضية محلية

🛠️ *أدوات CTF الأساسية*
`nmap` `gobuster` `burpsuite` `sqlmap`
`john` `hashcat` `binwalk` `strings`
`wireshark` `ghidra` `gdb` `pwndbg`

💡 *نصائح للمبتدئين*
• ابدأ بفئة Web ثم Crypto
• اقرأ WriteUps بعد الحل
• وثّق حلولك في مدونة أو GitHub
• انضم لـ Discord الخاص بالمنصة
• حل 1 تحدي يومياً على الأقل
""", parse_mode="Markdown")

# ══════════════════════════════════════════════
#  /about
# ══════════════════════════════════════════════
async def about_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🟢 *GreenHack Bot v3.0*\n\n"
        "بوت تيليغرام احترافي للأمن السيبراني والبرمجة\n\n"
        "🎯 *الهدف:* التعليم والتدريب في مجال السيبراني\n"
        "🛠️ *الأوامر:* 20+ أمر متكامل\n"
        "⚠️ *تحذير:* جميع الأدوات للأغراض التعليمية فقط\n\n"
        "اكتب `/help` لقائمة الأوامر الكاملة",
        parse_mode="Markdown"
    )

# ══════════════════════════════════════════════
#  Inline Keyboard
# ══════════════════════════════════════════════
MENU_DATA = {
    "m_net":      "🔍 *فحص الشبكة*\n\n`/ping` `/portscan` `/dns` `/ssl` `/myip`",
    "m_crypto":   "🔐 *التشفير*\n\n`/hash` `/b64enc` `/b64dec` `/rot13` `/hex` `/unhex` `/morse` `/unmorse`",
    "m_web":      "🌐 *فحص الويب*\n\n`/ssl` `/dns` `/myip`",
    "m_code":     "💻 *أدوات المبرمج*\n\n`/passgen` `/uuid` `/regex` `/encode`",
    "m_learn":    "📚 *تعليم سيبراني*\n\n`/owasp` `/roadmap` `/ctf` `/ports`",
    "m_glossary": "📖 *القاموس*\n\nاكتب `/glossary` لعرض جميع المصطلحات",
    "m_help":     "اكتب `/help` لعرض جميع الأوامر مع الشرح ✅",
}

async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    q = update.callback_query
    await q.answer()
    text = MENU_DATA.get(q.data, "غير معروف")
    await q.edit_message_text(
        text + "\n\n_اكتب الأمر في المحادثة للبدء_",
        parse_mode="Markdown"
    )

# ══════════════════════════════════════════════
#  رسائل عامة
# ══════════════════════════════════════════════
async def fallback(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "👋 مرحباً! اكتب `/help` لعرض الأوامر المتاحة\n"
        "أو `/start` للقائمة الرئيسية 🟢"
    )

# ══════════════════════════════════════════════
#  MAIN
# ══════════════════════════════════════════════
def main():
    if BOT_TOKEN == "YOUR_BOT_TOKEN_HERE":
        print("❌ ضع توكن البوت في متغير BOT_TOKEN أولاً!")
        print("   احصل عليه من @BotFather في تيليغرام")
        sys.exit(1)

    log.info("🟢 GreenHack Bot يبدأ...")
    app = Application.builder().token(BOT_TOKEN).build()

    handlers = [
        ("start",    start),
        ("help",     help_cmd),
        ("ping",     ping_cmd),
        ("portscan", portscan_cmd),
        ("dns",      dns_cmd),
        ("ssl",      ssl_cmd),
        ("myip",     myip_cmd),
        ("hash",     hash_cmd),
        ("b64enc",   b64enc_cmd),
        ("b64dec",   b64dec_cmd),
        ("rot13",    rot13_cmd),
        ("hex",      hex_cmd),
        ("unhex",    unhex_cmd),
        ("morse",    morse_cmd),
        ("unmorse",  unmorse_cmd),
        ("encode",   encode_cmd),
        ("passgen",  passgen_cmd),
        ("uuid",     uuid_cmd),
        ("regex",    regex_cmd),
        ("glossary", glossary_cmd),
        ("ports",    ports_cmd),
        ("owasp",    owasp_cmd),
        ("roadmap",  roadmap_cmd),
        ("ctf",      ctf_cmd),
        ("about",    about_cmd),
    ]

    for cmd, fn in handlers:
        app.add_handler(CommandHandler(cmd, fn))

    app.add_handler(CallbackQueryHandler(button_handler))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, fallback))

    log.info(f"✅ {len(handlers)} أمر مسجّل — البوت يعمل الآن!")
    app.run_polling(allowed_updates=Update.ALL_TYPES)

if __name__ == "__main__":
    main()