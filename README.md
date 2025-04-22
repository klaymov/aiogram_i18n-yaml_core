# aiogram_i18n-yaml_core

Installation:
```pip install - r requirements.txt```

Or use:
```pip install aiogram aiogram_i18n PyYAML```

```python
import asyncio
from contextlib import suppress
from logging import basicConfig, INFO
from typing import Any

from aiogram import Bot, Dispatcher, Router
from aiogram.client.default import DefaultBotProperties
from aiogram.enums import ParseMode
from aiogram.filters import CommandStart
from aiogram.types import Message

from aiogram_i18n import I18nContext, LazyProxy, I18nMiddleware, LazyFilter
from aiogram_i18n.types import (
    ReplyKeyboardMarkup, KeyboardButton
)

from i18n_yaml_core import YamlCore


router = Router(name=__name__)
rkb = ReplyKeyboardMarkup(
    keyboard=[[KeyboardButton(text=LazyProxy("help"))]],
    resize_keyboard=True
)


@router.message(CommandStart())
async def cmd_start(msg: Message, i18n: I18nContext) -> Any:
    name = msg.from_user.mention_html()
    return msg.reply(
        text=i18n.get("hello", user=name), # or i18n.hello(user=name)
        reply_markup=rkb
    )


@router.message(LazyFilter("help"))
async def cmd_help(msg: Message, i18n: I18nContext) -> Any:
    return msg.reply(text="-- " + i18n.get("start_info", 'uk') + " --") # you should pass the value for locale as a positional argument, not as a keyword


async def main() -> None:
    basicConfig(level=INFO)
    bot = Bot("42:ABC", default=DefaultBotProperties(parse_mode=ParseMode.HTML))

    i18n_middleware = I18nMiddleware(
        core=YamlCore(
            path="locales",
            default_locale="en"
        )
    )
    dp = Dispatcher()
    dp.include_router(router)
    i18n_middleware.setup(dispatcher=dp)

    await dp.start_polling(bot)


if __name__ == "__main__":
    with suppress(KeyboardInterrupt):
        asyncio.run(main())
```
