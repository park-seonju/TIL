- Data를 두 개 이상의 컴포넌트에서 사용하고자 하면 그 두 개의 공통 부모에서 선언해줘야함 ( 최소 공통 부모 )

```javascript
<script>
import Parent from '@/components/Parent.vue'

export default {
  name: 'App',
  components:{
    Parent,
  },
  data () {
    return {
      appData: '',
    }
  },
}
</script>
```

- 부모에서 자식으로 데이터를 넘길때 선언한 곳에다가 어떤 이름으로 넘겨줄지 알려줘야함 . (부모 부분)

```html
<template>
  <div id="app">
    <h2>App</h2>
    <input type="text" v-model="appData">
    <p>ParentData : 123</p>
    <Parent appsdata/> <!-- 이부분 -->
  </div>
</template>
```

- 자식에선 받을때 props를 설정해줘야 한다. ( 자식 부분 )

```html
<template>
  <div id="parent">
    <h2>Parent</h2>
    <input type="text">
    <p>appData : {{ appData }}</p>
  </div>
</template>

<script>
export default {
  props: {
    // appData: String,
    appData: {
      type: String,
    },
  },
}
</script>
```

- 자식에 데이터를 부모에게 보낼때도 부모에 데이터 선언

```javascript
<script>
import Parent from '@/components/Parent.vue'

export default {
  name: 'App',
  components:{
    Parent,
  },
  data () {
    return {
      appData: '',
      parentData: '',
    }
  },
}
</script>
```

- 단 부모 컴포넌트에 선언된 데이터를 자식에서 수정해봤자 적용되지 않는다.  고로 자식에서 data를 수정했다고 요청을 보냄!
  - **by 이벤트를 발생시켜서 요청을 보낸다**

- 속성에다가 쓸땐 **:** 바인딩 해야하고 그냥 태그안에 쓸땐 **{{}}** 사용가능