These are my notes from the Egghead.io course ["React Testing Cookbook"](https://egghead.io/series/react-testing-cookbook).

# 1 - Setting up dependencies

* `npm install --save-dev mocha expect react-addon-test-utils`

(Not `chai`, hmm...)

# 2 - Running tests

* In `package.json`, add `test` under `scripts`:

```sh
mocha './src/**/*.spec.js' --compilers js:babel-core/register
```

* In a spec file, we use `mocha` and `expect`:

```js
import expect from 'expect';

describe('emtpy', () => {
  it('should work', () => {
    expect(true).toEqual(true);
  });
});
```

# 3 - Utility modules

We have a hypothetical utility function (not React related) called `createId`, which generates unique IDs for a quote-keeper app.

```js
describe('createId', () => {
  it('should convert a description into a unique id', () => {
    const actual = createId(123, 'Cool example');
    const expected = '123-cool-example';
    expect(actual).toEqual(expected);
  });
});
```

# 4 - Intro to shallow rendering

```js
import React from 'react';
import TestUtils from 'react-addon-test-utils';
import expect from 'expect';

const CoolComponent = ({greeting}) => (
  <div>
    <h1>Greeting</h1>
    <div>{greeting}</h1>
  </div>
);

describe('CoolComponent', () => {
  it('should...', () => {
    // shallow rendering means only one component level deep
    const renderer = TestUtils.createRenderer();

    // same as ReactDOM.render()
    renderer.render(<CoolComponent greeting='hello world' />);

    // object output of shallow render
    const output = renderer.getRenderOutput();

    console.log(output);
  });
});
```

# 5 - JSX error diffs

```js
import React from 'react';
import TestUtils from 'react-addon-test-utils';
import expect from 'expect';

const CoolComponent = ({greeting}) => (
  <div>
    <h1>Greeting</h1>
    <div>{greeting}</h1>
  </div>
);

describe('CoolComponent', () => {
  it('should render the greeting', () => {
    const renderer = TestUtils.createRenderer();
    renderer.render(<CoolComponent greeting='hello world' />);
    const actual = renderer.getRenderOutput();
    const expected = (
      <div>
        <h1>Greeting</h1>
        <div>hello world</div>
      </div>
    );
    expect(actual).toEqual(expected)
  });
})
```

* What about when the test fails? We get a giant diff dump of internal react structure. We need better diffing of JSX output.

We can get this with [expect-jsx](https://github.com/algolia/expect-jsx).

Some things we can do with `expect-jsx`: `toEqualJSX`, `toNotEqualJSX`, `toIncludeJSX` (recursive children search). This last one means you don't have to tightly couple test-selectors to your HTML structure.

```sh
npm install --save-dev expect-jsx
```

```js
import React from 'react';
import TestUtils from 'react-addon-test-utils';
import expect from 'expect';
import expectJSX from 'expect-jsx';
expect.extend(expectJSX);

const CoolComponent = ({greeting}) => (
  <div>
    <h1>Greeting</h1>
    <div>{greeting}</h1>
  </div>
);

describe('CoolComponent', () => {
  it('should render the greeting', () => {
    const renderer = TestUtils.createRenderer();
    renderer.render(<CoolComponent greeting='hello world' />);
    const actual = renderer.getRenderOutput();
    const expected = <div>hello world</div>;

    expect(actual).toIncludeJSX(expected);
  });
})
```

# 6 - Element types with Shallow Rendering

This is about using the `type` attribute of `getRenderOutput()`. We can assert that the rendered output is a certain type of tag.

```js
import React from 'react';
import TestUtils from 'react-addon-test-utils';
import expect from 'expect';
import LikeCounter from './LikeCounter';

describe('LikeCounter', () => {
  it('should be a link', () => {
    const renderer = TestUtils.createRenderer();
    renderer.render(<LikeCounter count={5} />);

    const actual = renderer.getRenderOutput().type;
    const expected = 'a';
    expect(actual).toEqual(expected);

  });
});

```

# 7 - className with shallow rendering

We want to write tests to ensure our icons are rendering correctly.

```js
import React from 'react';
import TestUtils from 'react-addon-test-utils';
import expect from 'expect';
import Icon from './Icon';

describe('Icon', () => {
  it('should render the icon', () => {
    const renderer = TestUtils.createRenderer();
    renderer.render(<Icon name='facebook' />);

    // `includes` is an ES6 String.prototype function
    const actual = renderer.getRenderOutput().props.className.includes('facebook');
    const expected = true;

    expect(actual).toEqual(expected);

  });
});
```

# 8 - Conditional className with shallow rendering

Same idea as #7, but with a conditional on the component (`isActive=[bool]`). Test both true and false.

```js
import React from 'react';
import TestUtils from 'react-addon-test-utils';
import expect from 'expect';
import LikeCounter from './LikeCounter';

describe('LikeCounter', () => {
  it('should show the like count as active', () => {
    const renderer = TestUtils.createRenderer();
    renderer.render(<LikeCounter count={5} isActive={true} />);

    const actual = renderer.getRenderOutput().props.className.includes('LikeCounter--active');
    const expected = true;
    expect(actual).toEqual(expected);
  });

  it('should show the like count as inactive', () => {
    const renderer = TestUtils.createRenderer();
    renderer.render(<LikeCounter count={5} isActive={false} />);

    const actual = renderer.getRenderOutput().props.className.includes('LikeCounter--active');
    const expected = false;
    expect(actual).toEqual(expected);
  });
});

```

# 9 - Reusing test boilerplate

Let's refactor #8 using some shared logic in the describe block. He calls it a factory function. OK.

```js
import React from 'react';
import TestUtils from 'react-addon-test-utils';
import expect from 'expect';
import LikeCounter from './LikeCounter';

describe('LikeCounter', () => {
  function renderLikeCounter(isActive) {
    const renderer = TestUtils.createRenderer();
    renderer.render(<LikeCounter count={5} isActive={isActive} />);

    return renderer
      .getRenderOutput()
      .props
      .className
      .includes('LikeCounter--active');
  }

  describe('isActive', () => {
    it('should show the like count as active', () => {
      expect(renderLikeCounter(true)).toEqual(true);
    });

    it('should show the like count as inactive', () => {
      expect(renderLikeCounter(false)).toEqual(false);
    });
  });

});

```

# 10 - Children with shallow rendering

```js
import React from 'react';
import TestUtils from 'react-addon-test-utils';
import expect from 'expect';
import expectJSX from 'expect-jsx';
expect.extend(expectJSX);
import LikeCounter from './LikeCounter';

describe('LikeCounter', () => {
  it('should render like counts', () => {
    const renderer = TestUtils.createRenderer();
    renderer.render(<LikeCounter count={5} />);

    // const children = renderer.getRenderOutput().props.children;
    // We could just keep chaining .props.children ... on and on
    // But this is ugly. How else to do it? (A: toIncludeJSX)

    const expected = '5 likes';
    const actual = renderer.getRenderOutput();
    expect(actual).toIncludeJSX(expected);
  });
});

```

# 11 - The Redux Store - Multiple Actions

In this sort-of-integration-test we dispatch multiple actions, and only make one assertion at the end to verify the final state.

```js
import { store } from './store';
import expect from 'expect';

describe('store', () => {
  it('should work with a series of actions', () => {
    const actions = [
      {
        type: 'ADD_QUOTE_BY_ID',
        payload: {
          text: 'The best way to cheer yourself up is to try to cheer someone else up.',
          author: 'Mark Twain',
          id: 1,
          likeCount: 24
        }
      },
      {
        type: 'ADD_QUOTE_BY_ID',
        payload: {
          text: 'Whatever you are, be a good one.',
          author: 'Abraham Lincoln',
          id: 2,
          likeCount: 0
        }
      },
      {
        type: 'REMOVE_QUOTE_BY_ID',
        payload: { id: 1 }
      },
      {
        type: 'LIKE_QUOTE_BY_ID',
        payload: { id: 2 }
      },
      {
        type: 'LIKE_QUOTE_BY_ID',
        payload: { id: 2 }
      },
      {
        type: 'UNLIKE_QUOTE_BY_ID',
        payload: { id: 2 }
      },
      {
        type: 'UPDATE_THEME_COLOR',
        payload: { color: '#777777' }
      }
    ];

    actions.forEach(action => store.dispatch(action));

    const actual = store.getState();
    const expected = {
      quotes: [
        {
         text: 'Whatever you are, be a good one.',
         author: 'Abraham Lincoln',
         id: 2,
         likeCount: 1
        }
      ],
      theme: {
        color: '#777777'
      }
    };
    expect(actual).toEqual(expected);
  });
});

```

# 12 - The Redux Store - Initial state

In Redux, reducers must provide an initial/default state.

```js
import { store } from './store';
import expect from 'expect';

describe('store', () => {
  it('should initialize', () => {
    const actual = store.getState();
    const expected = {
      quotes: [],
      theme: { color: '#ffffff' }
    };
    expect(actual).toEqual(expected);
  });
});
```

# 13 - Redux Testing - Redux reducers

```js
import expect from 'expect';
import themeReducer from './themeReducer';

describe('themeReducer', () => {
  function stateBefore() {
    return {
      color: '#ffffff'
    };
  }

  const action = {
    type: 'UPDATE_THEME_COLOR',
    payload: { color: '#56ddff' }
  }

  it('should change the theme color', () => {
    const actual = themeReducer(stateBefore(), action)

    const actual = store.getState();
    const expected = { color: '#56ddff' }
    expect(actual).toEqual(expected);
  });
});

```

The use of `stateBefore` as a function is overkill here since it's just a single action. Let's try another reducer to go a bit further:


```js
import expect from 'expect';
import quoteReducer from './quoteReducer';

describe('quoteReducer', () => {
  function stateBefore() {
    return [
      {
        text: 'Lorem ipsum',
        author: 'Jane Doe',
        id: 1,
        likeCount: 7
      },
      {
        text: 'Ullamco laboris nisi ut aliquip',
        author: 'John Smith',
        id: 2,
        likeCount: 0
      },
    ]
  };



  it('should add quotes by id', () => {
    const action = {
      type: 'ADD_QUOTE_BY_ID',
      payload: {
        text: 'This is a new quote',
        author: 'Someone awesome',
        id: 3,
        likeCount: 0
      }
    }

    const actual = quoteReducer(stateBefore(), action)
    const expected = [
      {
        text: 'Lorem ipsum',
        author: 'Jane Doe',
        id: 1,
        likeCount: 7
      },
      {
        text: 'Ullamco laboris nisi ut aliquip',
        author: 'John Smith',
        id: 2,
        likeCount: 0
      },
      {
        text: 'This is a new quote',
        author: 'Someone awesome',
        id: 3,
        likeCount: 0
      }
    ];
    expect(actual).toEqual(expected);
  });

  it('should return prev state when trying to make likeCount negative', () => {
    const action = {
      type: 'UNLIKE_QUOTE_BY_ID',
      payload: { id: 2 }
    };
    const actual = quoteReducer(stateBefore(), action);
    const expected = stateBefore();
    expect(actual).toEqual(expected);
  });
});

```
