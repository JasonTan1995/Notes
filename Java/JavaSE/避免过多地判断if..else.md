## 避免过多地判断if..else


### 1. 使用Map.(用空间换取时间)

使用Map的话,会在加载时多消耗点时间,而且内存也会相应的增加.但在用户体验方面会提升很大.

- 修改前:如果我还想添加多几个错误异常呢?那岂不是要一直判断下去.
```java
if (StringUtils.equals(error, LOGIN_ERROR.LOCK.name())) {
        model.addAttribute(PageMsgController.ERROR_MSG, LOGIN_ERROR.LOCK.getMsg());
    } else if (StringUtils.equals(error, LOGIN_ERROR.BAD_CREDENTIALS.name())) {
        model.addAttribute(PageMsgController.ERROR_MSG, LOGIN_ERROR.BAD_CREDENTIALS.getMsg());
    } else if (StringUtils.equals(error, LOGIN_ERROR.FORBIDDEN.name())) {
        model.addAttribute(PageMsgController.ERROR_MSG, LOGIN_ERROR.FORBIDDEN.getMsg());
    } else if (StringUtils.equals(error, LOGIN_ERROR.TIMEOUT.name())) {
        model.addAttribute(PageMsgController.ERROR_MSG, LOGIN_ERROR.TIMEOUT.getMsg());
    } else if (StringUtils.equals(error, LOGIN_ERROR.LOGIN.name())) {
        model.addAttribute(PageMsgController.ERROR_MSG, LOGIN_ERROR.LOGIN.getMsg());
    } else if (StringUtils.equals(error, LOGIN_ERROR.INSUFFICIENT_AUTH.name())) {
        model.addAttribute(PageMsgController.ERROR_MSG, LOGIN_ERROR.INSUFFICIENT_AUTH.getMsg());
    } else if (StringUtils.equals(error, LOGIN_ERROR.HELP.name())) {
        model.addAttribute(PageMsgController.ERROR_MSG, LOGIN_ERROR.HELP.getMsg());
    } else {
        model.addAttribute(PageMsgController.ERROR_MSG, LOGIN_ERROR.UNKNOWN.getMsg());
    }
```

- 修改后:

```java
model.addAttribute(PageMsgController.ERROR_MSG, LOGIN_ERROR.getValue(error));
```

- 事先用Map把数据都装载起来.
```java
public static final Map<String, String> ERROR_LIST = Collections.unmodifiableMap(Arrays.stream(LOGIN_ERROR.values())
            .collect(Collectors.toMap(
                    constant -> constant.name(),
                    constant -> constant.msg
            )));

    public enum LOGIN_ERROR {
        LOCK("Account deactivated. Please contact admin."),
        BAD_CREDENTIALS("Invalid login credential."),
        FORBIDDEN("Please login again. (Timeout/Forbidden)."),
        TIMEOUT("Please login again. (Timeout)"),
        LOGIN("Please login."),
        NO_AUTH("Please login. (Auth not found)"),
        INSUFFICIENT_AUTH("Insufficient Authority."),
        HELP("Please contact system support."),
        UNKNOWN("Login error."),
        NO_PERMITTED("Not permitted to logon,Please contact administrator."),
        PWD_EXPIRED("Password has expired. Please change your password."),
        DISABLED("Your account is disabled/expired. Please contact administrator."),
        CHANGE_PWD("Please change your password.");

        private String msg;

        LOGIN_ERROR(String msg) {
            this.msg = msg;
        }

        public String getMsg() {
            return msg;
        }

        public String getUri() {
            return LOGIN_ERROR_PARA_URI + this.name();
        }

        public static String getValue(String name) {
            return StringUtils.isEmpty(ERROR_LIST.get(name)) ? UNKNOWN.msg : ERROR_LIST.get(name);
        }
    }
```
