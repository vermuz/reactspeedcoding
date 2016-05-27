# Redux State Container (X)

When designing React apps, UI state becomes an important concern. How state is managed
across your component hierarchy during your app lifecycle gets complex fast as
the number of components and user interactions increase.

React is the *View* for your app. We need an equally elegant solution for
managing the *State* and *Data Flow* within your app. This becomes even more important
concern for Single Page Apps as efficient state management leads to performant
user experience.

Fortunately Facebook has already thought this through for us. They have introduced
to the open source, [Flux application architecture][3] for building user interfaces.

A> Flux is the application architecture that Facebook uses for building client-side web applications.
A> It complements React's composable view components by utilizing a unidirectional data flow.
A> It's more of a pattern rather than a formal framework...
A> Flux applications have three major parts: the dispatcher, the stores, and the views (React components).
A> -- http://facebook.github.io/flux/

Redux is a very popular library in the React ecosystem.
It evolves from Flux design patterns and is based on [Elm architecture][2] which is a simple
pattern for infinitely nested components.

{pagebreak}

## The Roadmap app

To help understand this important chapter, let us create a relatively complex app
to manage the the roadmap for ReactSpeed book and companion code. We want to list
upcoming, recent content and code features. Users should have the capability
to *Like* features they want to see first or jump to live feature demos and content.

![Roadmap app wireframe](images/roadmap.jpg)

Our roadmap app will require a Feature component to render the individual feature.
It will also require a FeatureList component to manage list of feature components. We will
add a FeatureSearch component to search listed features.
A CategoryFilter component will list features by categories like
components, styles, chapters, sections, and strategies.

You will note that various components within this app will interact with each
other (blue dashed lines in the wireframe). Changing filters will interact with search,
reducing the scope of what can be searched. Search will interact with features, showing only features
that match the text entered in search. Number of likes will interact with order of features.

Our app will also maintain several UI states. Some candidate states could be,
active filter, order of features, search text, and number of *Likes*.

{pagebreak}

## Redux basics

Redux is remarkably simple API and an elegant architectural design pattern at the same time.

Redux introduces three important and inter-related concepts.

**Stores for your state data.**

- All your app state is a single data structure.
- Think a single JSON document representing the entire state of your app.
- State is immutable.
- Creating new state makes a copy of existing state with the new changes.

**Actions for managing data flow.**

- Actions usually define functions with two arguments. First, type of state change, and
second, change data.
- Type of state change is usually defined as strings or constants representing strings,
named after the action performed on the state tree leading to the data change.
- The data change can be represented as a single value, an object with multiple values,
or can even be implicitly understood from the action type. Examples: INCREMENT_LIKES,
TOGGLE_TODO.
- Actions are the only way to communicate change within the state tree.

**Reducer functions.**

- Reducers are written as pure functions.
- Reducers manipulate state based on Actions.
- Fixed input to the reducer function are two arguments. First, the existing state tree, and
second, the action to be applied on the state tree.
- Reducer returns a predictable output as the new state tree after applying the action.
- As reducer is a pure function, it should not have any unpredictable behavior including
data manipulation based on random functions or timestamp. Input should determine the output predictably.

We will discover other advanced concepts that elaborate the three core concepts, as we go through
coding the Roadmap app in the Redux way.

{pagebreak}

## State tree definition

Redux is all about managing state and data flow within our app. A good first
step is to define the state tree for our app. Redux treats the entire state
of your app as single data structure. Think of the state tree as a JSON document
describing your app.

Drawing this JSON document for our Roadmap app will make things clearer.

We will maintain state for list of features with feature title, category, and likes count.
We will store text entered by user in search box. We will also capture category
selected by the user for filtering features.

When our app starts the initial state has defaults set for various
states maintained by our app.

{title="Roadmap state tree, initial state", lang=json}
~~~~~~~
{
  features: [],
  searchText: '',
  categoryFilter: 'SHOW_ALL'
}
~~~~~~~

New features can be added to our app, updating the features list within
our state tree.

{title="Roadmap state tree, new features", lang=json}
~~~~~~~
{
  features: [
    { title: 'Navigation', likes: 0, category: 'COMPONENT' },
    { title: 'Redux state container', likes: 0, category: 'CHAPTER' },
    { title: 'Roadmap', likes: 0, stage: 'APP' }
  ],
  searchText: '',
  categoryFilter: 'SHOW_ALL'
}
~~~~~~~

As our users ```like``` the features the respective likes count increments.

{title="Roadmap state tree, trending", lang=json}
~~~~~~~
{
  features: [
    { title: 'Navigation', likes: 0, category: 'COMPONENT' },
    { title: 'Redux state container', likes: 2, category: 'CHAPTER' },
    { title: 'Roadmap', likes: 1, stage: 'APP' }
  ],
  searchText: '',
  categoryFilter: 'SHOW_ALL'
}
~~~~~~~

As users select a new category filter or enter search text, respective state
gets updated.

{title="Roadmap state tree, live", lang=json}
~~~~~~~
{
  features: [
    { title: 'Navigation', likes: 0, category: 'COMPONENT' },
    { title: 'Redux state container', likes: 2, category: 'CHAPTER' },
    { title: 'Roadmap', likes: 1, stage: 'APP' }
  ],
  searchText: 'new search text',
  categoryFilter: 'SHOW_CHAPTERS'
}
~~~~~~~

We can continue to evolve our state tree by adding other states for our app. Ideally
you would do this as you evolve the design of your app. For now we can move to the next
stage quickly.

{pagebreak}

## Redux specification

Before jumping into creating our Redux app, we can specify how the Redux stores, actions,
and reducers behave as the state tree changes within our app. We can do so using our
test environment setup in **Test App Components** chapter.

{title="03_roadmap.spec.js test suite spec", lang=javascript}
~~~~~~~
import { describe, it } from 'mocha';

describe('Roadmap Redux Spec', () => {
  it('should get initial state for store');
  it('should add first feature of COMPONENT category');
  it('should initialize first feature with default state');
  it('should increment likes count for first feature');
  it('should set a new categoryFilter');
  it('should add second feature of CHAPTER category');
  it('should add third feature of APP category');
  it('should set new search text');
});
~~~~~~~

When we run this test using ```npm run test``` we notice following test results.

{title="Terminal output running Roadmap test suite", lang=text}
~~~~~~~
...

Roadmap Redux Spec
  - should get initial state for store
  - should add first feature of COMPONENT category
  - should initialize first feature with default state
  - should increment likes count for first feature
  - should set a new categoryFilter
  - should add second feature of CHAPTER category
  - should add third feature of APP category
  - should set new search text

~~~~~~~

Redux apps are best designed in the order Actions > Reducers > Store > Components.
So it is a good idea to develop tests for our Redux app as we evolve the app, through
these stages. We do not have to wait for the components to render to see results of
our Redux design.

{pagebreak}

## Actions for Roadmap

Actions in Redux represent data payloads sent between your app and the Redux store.
Actions are the only way you can update your state tree within the store.

We can derive some Redux Actions from the state tree we just defined.

{title="/actions/roadmap.js", lang=javascript}
~~~~~~~
/*
 * action types
 */

export const ADD_FEATURE = 'ADD_TODO';
export const SET_CATEGORY_FILTER = 'SET_CATEGORY_FILTER';
export const LIKE_FEATURE = 'LIKE_FEATURE';
export const SEARCH_TEXT = 'SEARCH_TEXT';
/*
 * other constants
 */

export const CategoryFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPONENTS: 'SHOW_COMPONENTS',
  SHOW_CHAPTERS: 'SHOW_CHAPTERS'
};

export const Categories = {
  CHAPTER: 'CHAPTER',
  COMPONENT: 'COMPONENT',
  APP: 'APP'
};

/*
 * action creators
 */

export function addFeature(title, category) {
  return {
    type: ADD_FEATURE,
    category,
    title };
}

export function setCategoryFilter(filter) {
  return { type: SET_CATEGORY_FILTER, filter };
}

export function setSearchText(text) {
  return { type: SEARCH_TEXT, text };
}

export function likeFeature(index) {
  return { type: LIKE_FEATURE, index };
}
~~~~~~~

Notice that we have defined constants for our state values including categories
and category filters. We are maintaining consistency with definition of our action types
as string constants. This is not a Redux requirement.

{pagebreak}

## Reducers for Roadmap

Now that we have some actions defined, we can write what happens to our state
tree when these actions are called. We do this writing pure functions called
reducers which take two arguments, the current state tree and the action to
perform on the state tree. The reducer then returns the new state tree.

{title="/app/reducers/roadmap.js", lang=javascript}
~~~~~~~
import * as actions from '../actions/roadmap';

const initialState = {
  searchText: '',
  categoryFilter: actions.CategoryFilters.SHOW_ALL,
  features: []
};

export default function roadmapApp(state = initialState, action) {
  switch (action.type) {
  case actions.LIKE_FEATURE:
    return Object.assign({}, state, {
      features: state.features.map((feature, index) => {
        if (index === action.index) {
          return Object.assign({}, feature, {
            likes: feature.likes + 1
          });
        }
        return feature;
      })
    });
  case actions.SEARCH_TEXT:
    return Object.assign({}, state, {
      searchText: action.text
    });
  case actions.SET_CATEGORY_FILTER:
    return Object.assign({}, state, {
      categoryFilter: action.filter
    });
  case actions.ADD_FEATURE:
    return Object.assign({}, state, {
      features: [
        ...state.features,
        {
          title: action.title,
          category: action.category,
          likes: 0
        }
      ]
    });
  default:
    return state;
  }
}
~~~~~~~

The ```Object.assign()``` method creates copy of existing state into new state while updating
the new changes suggested by the action.

{pagebreak}

Now that we have defined the Reducer and Action, it is time to create the Store.

This is the simplest part of writing a Redux app.

{title="/app/store/roadmap.js", lang=javascript}
~~~~~~~
import { createStore } from 'redux';
import roadmapApp from '../reducers/roadmap';
const store = createStore(roadmapApp);
export default store;
~~~~~~~

We have exported our store so that we can use it within other parts of our app including
our test suite.

{pagebreak}

## Test store, actions, and reducers

Let us elaborate our test spec with assertions to test our Redux app so far.

{title="03_roadmap.spec.js test suite spec", lang=javascript}
~~~~~~~
import { describe, it } from 'mocha';
import { expect } from 'chai';
import store from '../../app/store/roadmap';
import * as actions from '../../app/actions/roadmap';

describe('Roadmap Redux', () => {
  it('should get initial state for store', () => {
    expect(store.getState().features.length).to.equal(0);
    expect(store.getState().categoryFilter)
      .to.equal(actions.CategoryFilters.SHOW_ALL);
    expect(store.getState().searchText)
      .to.equal('');
  });
  it('should add first feature of COMPONENT category', () => {
    store.dispatch(
      actions.addFeature('New Component Feature', actions.Categories.COMPONENT)
    );
    expect(store.getState().features.length).to.equal(1);
    expect(store.getState().features[0].category)
      .to.equal(actions.Categories.COMPONENT);
  });
  it('should initialize first feature with default state', () => {
    expect(store.getState().features[0].likes).to.equal(0);
  });
  it('should increment likes count for first feature', () => {
    store.dispatch(actions.likeFeature(0)); // likes = 1
    store.dispatch(actions.likeFeature(0)); // likes = 2
    expect(store.getState().features[0].likes).to.equal(2);
  });
  it('should set a new categoryFilter', () => {
    expect(store.getState().categoryFilter)
      .to.equal(actions.CategoryFilters.SHOW_ALL);
    store.dispatch(actions
      .setCategoryFilter(actions.CategoryFilters.SHOW_COMPONENTS));
    expect(store.getState().categoryFilter)
      .to.equal(actions.CategoryFilters.SHOW_COMPONENTS);
  });
  it('should add second feature of CHAPTER category', () => {
    store.dispatch(
      actions.addFeature('Second Chapter Feature', actions.Categories.CHAPTER)
    );
    expect(store.getState().features.length).to.equal(2);
    expect(store.getState().features[1].category)
      .to.equal(actions.Categories.CHAPTER);
  });
  it('should add third feature of APP category', () => {
    store.dispatch(
      actions.addFeature('Third App Feature', actions.Categories.APP)
    );
    expect(store.getState().features.length).to.equal(3);
    expect(store.getState().features[2].category)
      .to.equal(actions.Categories.APP);
  });
  it('should set new search text', () => {
    expect(store.getState().searchText)
      .to.equal('');
    store.dispatch(actions
      .setSearchText('new search text'));
    expect(store.getState().searchText)
      .to.equal('new search text');
  });
});
~~~~~~~

When we run this test using ```npm run test``` we notice following test results.

{title="Terminal output running Roadmap test suite", lang=text}
~~~~~~~
...

Roadmap Redux Spec
  - should get initial state for store
  - should add first feature of COMPONENT category
  - should initialize first feature with default state
  - should increment likes count for first feature
  - should set a new categoryFilter
  - should add second feature of CHAPTER category
  - should add third feature of APP category
  - should set new search text

17 passing (149ms)

~~~~~~~

Now as we evolve our Redux app, we can continue adding to our test suite.

{pagebreak}

## Component specification

Let us write the specification for our Roadmap app using the BDD test environment
we setup using **Test App Components** chapter.

For this we will use Mocha only to start with and specify our component hierarchy
as a to-be-implemented test suite.

{title="03_roadmap.spec.js", lang=javascript}
~~~~~~~
import { describe, it } from 'mocha';

describe('<Roadmap />', () => {
  it('should create one .roadmap component');

  describe('<SearchFilter />', () => {
    it('should create one .search-filter component');

    describe('<FeatureSearch />', () => {
      it('should create one .feature-search component');
    });

    describe('<CategoryFilter />', () => {
      it('should create N .category-filter components');
    });
  });

  describe('<FeatureList />', () => {
    it('should create one .feature-list component');

    describe('<Feature />', () => {
      it('should create N .feature components');

      describe('<Category />', () => {
        it('should create one .category component per .feature');
      });

      describe('<Likes />', () => {
        it('should create one .likes component per .feature');
      });

      describe('<FeatureDetail />', () => {
        it('should create one .feature-detail component per .feature');
      });
    });
  });
});
~~~~~~~

When we run this test using ```npm run test``` we notice following test results.

{title="Terminal output running Roadmap test suite", lang=text}
~~~~~~~
...

  <Roadmap />
    - should create one .roadmap component
    <SearchFilter />
      - should create one .search-filter component
      <FeatureSearch />
        - should create one .feature-search component
      <CategoryFilter />
        - should create N .category-filter components
    <FeatureList />
      - should create one .feature-list component
      <Feature />
        - should create N .feature components
        <Category />
          - should create one .category component per .feature
        <Likes />
          - should create one .likes component per .feature
        <FeatureDetail />
          - should create one .feature-detail component per .feature


  9 passing (144ms)
  12 pending
~~~~~~~

Here are the strategies to consider when creating component specification for your app.

- **Component hierarchy.** Represent the component hierarchy starting
at the top level owner component traversing through child or owned components.
- **JSX naming.** Identify component name using JSX closing tag statement <Component /> even if
the component has props.
- **Class name.** The ```it``` spec statements can refer to className associated
with the component. This className can then be used by Enzyme find() method as well.
- **Cardinality.** Specify cardinality (zero, one, or many) of component(s) expected to be
created during normal use case of the application. This can be checked
in the test implementation.
- [TODO] Consider representing component state or props during specification stage
as additional ```it``` spec statements.
- [TODO] What aspects of state tree can be represented in the formal spec?

{pagebreak}

I> ## Chapter In Progress
I> We are still writing this chapter. Please watch this space for updates.
I> Plan is to add working samples to explain how Redux predictable state container
I> makes our app architecture more robust.

[1]: https://github.com/facebook/immutable-js
[2]: http://guide.elm-lang.org/architecture/index.html
[3]: http://facebook.github.io/flux/