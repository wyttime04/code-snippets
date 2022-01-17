# vscode Code Sinppets <!-- omit in toc -->
新專案一次要產太多東西，所以就弄了一些偷懶用的小東東

使用 [.vscode](/.vscode) code sinnpets 定義文件格式

- [Code Sinppets](#code-sinppets)
  - [readme](#readme)
    - [desaction](#desaction)
  - [rdx](#rdx)
    - [imaction](#imaction)
    - [rdxaction](#rdxaction)
    - [rdxreducer](#rdxreducer)
    - [rdxrdcase](#rdxrdcase)
    - [rdxcontainer](#rdxcontainer)
  - [api](#api)
    - [apiaction](#apiaction)
    - [apirdcase](#apirdcase)
    - [apirdmucase](#apirdmucase)
    - [imapi](#imapi)
    - [apifunction](#apifunction)
    - [apisaga](#apisaga)
    - [apicwsaga](#apicwsaga)
    - [apiworker](#apiworker)

# Code Sinppets

## readme
使用 (`fn` + ) `control` + `Space` 呼叫

### desaction
文件使用描述 Action 格式

~~~Markdown
###_Action Title (Action_Name)

<details>
<summary>
API

```
GET api/...
```
</summary>

- **Body**
  | KEY | TYPE | DESC |
  | --- | ---- | ---- |
  |     |      |      |

- **Resp**
  | KEY    | TYPE    | DESC         |
  | ------ | ------- | ------------ |
  | state  | Boolean | API 執行結果 |
  | errmsg | String  | 錯誤資訊     |
  | data   | null    |              |

</details>
~~~

---

## rdx
redux 相關基底格式

### imaction
`action/` 下的 action 根據路徑轉換 import 路徑

~~~javascript
import action from '../../lib/createAction'
~~~


### rdxaction
定義單一 action

~~~javascript
export const action_name = 'action_name'
/**
 * desc
 * @param {string} param1 參數
 */
export const action_name = data => action(action_name, data)
~~~


### rdxreducer
定義 reducer

~~~javascript
import * as actions from '../../actions'

const init = {
    data: {},
    isFetching: false,
    errmsg: '',
}

export default function (
    state = init, action) {
    switch (action.type) {
        // INIT_DATA
        case actions[`INIT_DATA`]:
            return init
    
        default:
            return state
    }
}
~~~

### rdxrdcase
定義 reducer 處理的 action case

~~~javascript
// ACTION_NAME
case actions[`ACTION_NAME`]:
    return {
        ...state,
    }
~~~

### rdxcontainer
container 基本配置，根據相對 `containers/` 的路徑 import 對應 component

~~~javascript
import { connect } from 'react-redux'
import index from 'components/Permission/User/index'
import {
    action_name
} from '../../../actions'


const mapStateToProps = (state) => {
    const { account } = state.user
    return {
        account,
    }
}

const mapDispatchToProps = {
    action_name
}

export default connect(mapStateToProps, mapDispatchToProps)(index)
~~~


---

## api
api 相關進階 redux 格式

### apiaction
定義 api 相關 action

~~~javascript
export const get_action_name = 'get_action_name'
/**
 * desc
 * @param {string} param1 參數
 */
export const get_action_name = data => action(get_action_name, data)

export const get_action_name_succ = data => action(get_action_name_SUCC, data)
export const get_action_name_SUCC = 'get_action_name_SUCC'

export const get_action_name_fail = data => action(get_action_name_FAIL, data)
export const get_action_name_FAIL = 'get_action_name_FAIL'
~~~

### apirdcase
用於 reducer 定義 api 相關 action case

~~~javascript
// GET_ACTION_NAME
case actions[`GET_ACTION_NAME`]:
    return {
        ...state,
        errmsg: '',
        isFetching: true,
    }
case actions[`GET_ACTION_NAME_SUCC`]:
    return {
        ...state,
        data: action.data,
        isFetching: false,
    }
case actions[`GET_ACTION_NAME_FAIL`]:
    return {
        ...state,
        errmsg: action.errmsg,
        data: action.data || init.data,
        isFetching: false,
    }
~~~

### apirdmucase
用於 reducer 定義不同 method 的 api 相關 action case

> 自行在外部先定義 `ACTION` 名稱，method 可選擇 `GET`, `POST`, `DELETE`, `PUT`, `PATCH`

~~~javascript
// POST
case actions[`POST_${ACTION}`]:
    return {
        ...state,
        errmsg: '',
        isFetching: true,
    }
case actions[`POST_${ACTION}_SUCC`]:
    return {
        ...state,
        data: action.data,
        isFetching: false,
    }
case actions[`POST_${ACTION}_FAIL`]:
    return {
        ...state,
        errmsg: action.errmsg,
        data: action.data || init.data,
        isFetching: false,
    }
~~~


### imapi
用於 `saga/`，協助 import 相關檔案、function

~~~javascript
import { take, call, put, select } from 'redux-saga/effects'
import * as actions from '../../actions'
import api from '../../lib/api'
~~~

### apifunction
定義 api function

~~~javascript
/**
 * desc
 * @param {string} param1 參數
 */
export function fn_name({type, ...data}) {
    return api({
        cmd: `User/Login`,
        method: 'GET',
        data
    })
}
~~~

### apisaga
於 `sagas/index.js` 定義 saga，使用 [commonWorker](/src/app/sagas/Worker.js) 定義流程

~~~javascript
    yield takeLatest(actions.ACTION_NAME, (action) => commonWorker(action, api.function))
~~~

### apicwsaga
於 `sagas/index.js` 定義 saga，使用自訂 worker 定義流程

~~~javascript
    yield takeLatest(actions.ACTION_NAME, api.function)
~~~

### apiworker
自訂 saga worker

~~~javascript
export function* fnName(action) {
    const actionType = action.type.toLowerCase()
    const res = yield call(api, action)
    const succ = actions[`${ actionType }_succ`]
    const fail = actions[`${ actionType }_fail`]
    if (res.ok) {
        yield put(succ(res.body))
        yield put(actions.enqueue_snackbar({
            message: `succ_msg`,
            key: new Date().getTime(),
            options: {
                variant: 'success'
            }
        }))
    }
    else {
        yield put(fail(res.body))
        yield put(actions.enqueue_snackbar({
            message: `errmsg`,
            key: new Date().getTime(),
            options: {
                variant: 'error'
            }
        }))
    }
}
~~~
