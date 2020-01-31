---
layout: post
title: React testing library
author: bubuta
categories: general
comments: true
---

[react-testing-library](https://github.com/testing-library/react-testing-library): Simple and complete React DOM testing utilities that encourage good testing practices.

create-react-app 으로 react app 생성 하고 나면 App.test.js 가 생성되어 있습니다.

```js
import React from 'react';
import { render } from '@testing-library/react';
import App from './App';
it('renders welcome message', () => {
  const { getByText } = render(<App />);
  expect(getByText('Learn React')).toBeInTheDocument();
});
```

이 부분이 react-testing-library import 하는 부분입니다.

```js
import { render } from '@testing-library/react';
```

> encourages better testing practices. Its primary guiding principle is:  
> The more your tests resemble the way your software is used, the more confidence they can give you.

위 예제 처럼 간단한 render test 붙여주는 것 부터 시작해 보면 좋을 것 같습니다.  

UI 개발 할때는 code 수정한 부분이 의도대로 잘 변경됬는지 확인할 때  
브라우저를 띄워서 button click 해보고 하는 과정들이 번거로울 때가 많았습니다.

```js
import React from 'react'
import {render, fireEvent, screen} from '@testing-library/react'
import HiddenMessage from '../hidden-message'

test('shows the children when the checkbox is checked', () => {
  const testMessage = 'Test Message'
  render(<HiddenMessage>{testMessage}</HiddenMessage>)
  fireEvent.click(screen.getByLabelText(/show/i))
  expect(screen.getByText(testMessage)).toBeInTheDocument()
}

```

이런 식으로 test case 만들어두면 Editor 와 browser를 덜 왔다 갔다 하고
CI / CD 에도 도움이 되니까 좋을 것 같습니다.
