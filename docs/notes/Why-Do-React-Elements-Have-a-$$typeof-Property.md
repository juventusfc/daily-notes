# Why Do React Elements Have a \$\$typeof Property 读后感

```javascript
{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'), // 🧐 Who dis
}
```

本篇博文主要讲了 React Element 中的 `$$typeof` 属性。

```javascript
<p>{message.text}</p>
```

不进行转义时，如果上面的代码 `message.text` 为 `<img>` 或 `<script>`，会存在 XSS 攻击。为了防止 XSS 攻击，React 默认会将字符串转义。

`$$typeof` 属性存在的真正原因时为了防止 Server 端问题引起的 XSS 攻击。由于 Symbol 的特殊情况，它不能放在 JSON 中，所以从 Server 返回过来的 JSON 中是没有 `$$typeof` 属性的。React 通过这个属性判断是 Client 还是 Server 产生的 Element。
