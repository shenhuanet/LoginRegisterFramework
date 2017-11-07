# LoginRegisterFramework
Almost every app will have login features, login, registr by mobile phone, forgotten password for these entrances, these features in the app how to achieve it? This module looks very simple, but to be done you need to consider many details, such as the user's input fault tolerance, the operation of the prompt copy settings, login successfully save the user information and so on.

## Business flowchart

![Business flowchart](https://github.com/shenhuanet/LoginRegisterFramework/blob/master/img_flow.png)


# Business logic description
1. Click to enter the personal center or user login status of the operation, first determine whether the user has logged in.
2. If you have already logged in, continue to the back of the business, otherwise, jump to the login page to log in.
3. If you already have an account, you can log in directly, or you can directly select the third-party platform authorized login.
4. If you do not register an account, you need to register the account first, and then log in successfully. Or you can log in without the account and authorize the third-party platform.
5. If you have an account, but forget the password, you need to reset the password, or directly login.

# Implementation
Login can login using an account, and now the app is basically a mobile phone number login, registration is also a mobile phone corresponds to an account, by sending verification code to verify; users can also choose third-party platform for login, usually provide WeChat, QQ , Sina microblogging mainstream social networking platform for authorized login, where I used the ** Umeng SDK ** to achieve.

## About Me
CSDN：http://blog.csdn.net/klxh2009<br>
JianShu：http://www.jianshu.com/u/12a81897d5bc

## License

    Copyright 2017 shenhuanet

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
