<!-- markdownlint-disable MD002 MD041 -->

В этом руководстве рассказывается о создании приложения Xamarin, которое использует API microsoft Graph для получения сведений о календаре для пользователя.

> [!TIP]
> Если вы предпочитаете просто скачать завершенный учебник, вы можете скачать или клонировать [GitHub репозиторий](https://github.com/microsoftgraph/msgraph-training-xamarin).

## <a name="prerequisites"></a>Предварительные требования

Перед началом этого руководства [](https://visualstudio.microsoft.com/vs/) необходимо установить Visual Studio на компьютере с Windows 10 с включенным режимом [разработчика.](https://docs.microsoft.com/windows/uwp/get-started/enable-your-device-for-development) Если у вас нет Visual Studio, посетите предыдущую ссылку для параметров загрузки.

Кроме того, в рамках установки Visual Studio Xamarin. Инструкции по установке и настройке [Xamarin](/xamarin/cross-platform/get-started/installation) см. в инструкции по установке Xamarin.

Кроме того, необходимо иметь доступ к Компьютеру Mac с Visual Studio для Mac установлен. Если у вас нет доступа, вы все равно можете завершить этот учебник, но не сможете завершить разделы, определенные для iOS.

Вы также должны иметь личную учетную запись Майкрософт с почтовым ящиком на Outlook.com или учетную запись Microsoft work или school. Если у вас нет учетной записи Майкрософт, существует несколько вариантов получения бесплатной учетной записи:

- Вы можете [зарегистрироваться для новой личной учетной записи Майкрософт.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Вы можете [зарегистрироваться в программе Office 365 разработчика,](https://developer.microsoft.com/office/dev-program) чтобы получить бесплатную Office 365 подписку.

> [!NOTE]
> Этот учебник был написан Visual Studio версии 16.10.3 и Visual Studio для Mac версии 8.5.1. На обеих машинах установлена платформа SDK Для Android 28. Действия в этом руководстве могут работать с другими версиями, но они не были проверены.

## <a name="feedback"></a>Отзывы

В репозитории [](https://github.com/microsoftgraph/msgraph-training-xamarin)GitHub.
