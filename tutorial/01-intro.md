<!-- markdownlint-disable MD002 MD041 -->

В этом руководстве рассказывается, как создать приложение Xamarin, использующее API Microsoft Graph для получения сведений календаря для пользователя.

> [!TIP]
> Если вы предпочитаете просто скачать завершенный учебник, вы можете скачать или клонировать [репозиторий GitHub.](https://github.com/microsoftgraph/msgraph-training-xamarin)

## <a name="prerequisites"></a>Предварительные условия

Перед началом работы с [](https://visualstudio.microsoft.com/vs/) этим учебником необходимо установить Visual Studio компьютере под управлением Windows 10 с [включенным режимом разработчика.](https://docs.microsoft.com/windows/uwp/get-started/enable-your-device-for-development) Если у вас нет Visual Studio, посетите предыдущую ссылку для скачивания.

Кроме того, в рамках установки Visual Studio Xamarin. Инструкции по установке и настройке [Xamarin](/xamarin/cross-platform/get-started/installation) см. в инструкциях по установке Xamarin.

Кроме того, у вас должен быть доступ к компьютеру Mac с установленным Visual Studio mac. Если у вас нет доступа, вы по-прежнему можете выполнить это руководство, но не сможете завершить разделы для iOS.

У вас также должна быть личная учетная запись Майкрософт с почтовым ящиком на Outlook.com или учетная запись Майкрософт для работы или учебного заведения. Если у вас нет учетной записи Майкрософт, существует несколько вариантов получения бесплатной учетной записи:

- Вы можете [зарегистрироваться для новой личной учетной записи Майкрософт.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Вы можете зарегистрироваться в программе для разработчиков [Office 365,](https://developer.microsoft.com/office/dev-program) чтобы получить бесплатную подписку на Office 365.

> [!NOTE]
> Это руководство было написано с Visual Studio 2019 версии 16.8.3 и Visual Studio для Mac версии 8.5.1. На обоих компьютерах установлена платформа SDK для Android 28. Действия в этом руководстве могут работать с другими версиями, но не были протестированы.

## <a name="feedback"></a>Отзывы

Поделитесь с этим учебником отзывами в [репозитории GitHub.](https://github.com/microsoftgraph/msgraph-training-xamarin)
