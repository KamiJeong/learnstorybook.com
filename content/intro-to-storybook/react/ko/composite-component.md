---
title: '합성 컴포넌트 조립하기'
tocTitle: '합성 컴포넌트'
description: '더 간단한 컴포넌트로 합성 컴포넌트 조립하기'
commit: '8db511e'
---

지난 챕터에서 우리는 첫 컴포넌트를 만들었습니다; 이번 챕터에서는 task 목록인 TaskList를 만드는 법을 배웁니다. 컴포넌트들을 함께 결합해 보고 더 복잡해때 어떤 일이 벌어지는지 봅시다.

## Tasklist

Taskbox는 고정된 task들을 기본 작업들 위에 배치함으로써 강조를 합니다. 그러면 `TaskList`의 두가지 변형인 기본 task들과 기본 및 고정된 task들을 위한 스토리들을 만들어야 합니다.

![default and pinned tasks](/intro-to-storybook/tasklist-states-1.png)

`Task` 데이터는 비동기적으로 전송 될 수 있으므로, 우리는 **또한** 연결이 없는 경우 렌링을 위한 로딩 상태가 필요합니다. 추가로, task들이 업을의 빈 상태가 필요합니다.

![empty and loading tasks](/intro-to-storybook/tasklist-states-2.png)

## 설정하기

복합 컴포넌트는 기본 컴포넌트들과 크게 다르지 않습니다. `TaskList` 컴포넌트와 스토리 파일 `src/components/TaskList.js` 와 `src/components/TaskList.stories.js`를 작성해 봅시다.

`TaskList`의 대략적인 구현으로 시작합시다. 우선 이전의 `Task` 컴포넌트를 가져와서 속성과 액선을 입력으로 전달해야 합니다.

```javascript
// src/components/TaskList.js

import React from 'react';

import Task from './Task';

function TaskList({ loading, tasks, onPinTask, onArchiveTask }) {
  const events = {
    onPinTask,
    onArchiveTask,
  };

  if (loading) {
    return <div className="list-items">loading</div>;
  }

  if (tasks.length === 0) {
    return <div className="list-items">empty</div>;
  }

  return (
    <div className="list-items">
      {tasks.map(task => (
        <Task key={task.id} task={task} {...events} />
      ))}
    </div>
  );
}

export default TaskList;
```

다음으로 `Tasklist`의 테스트 상태를 스토리 파일에 작성합시다.

```javascript
// src/components/TaskList.stories.js

import React from 'react';

import TaskList from './TaskList';
import { taskData, actionsData } from './Task.stories';

export default {
  component: TaskList,
  title: 'TaskList',
  decorators: [story => <div style={{ padding: '3rem' }}>{story()}</div>],
  excludeStories: /.*Data$/,
};

export const defaultTasksData = [
  { ...taskData, id: '1', title: 'Task 1' },
  { ...taskData, id: '2', title: 'Task 2' },
  { ...taskData, id: '3', title: 'Task 3' },
  { ...taskData, id: '4', title: 'Task 4' },
  { ...taskData, id: '5', title: 'Task 5' },
  { ...taskData, id: '6', title: 'Task 6' },
];

export const withPinnedTasksData = [
  ...defaultTasksData.slice(0, 5),
  { id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED' },
];

export const Default = () => <TaskList tasks={defaultTasksData} {...actionsData} />;

export const WithPinnedTasks = () => <TaskList tasks={withPinnedTasksData} {...actionsData} />;

export const Loading = () => <TaskList loading tasks={[]} {...actionsData} />;

export const Empty = () => <TaskList tasks={[]} {...actionsData} />;
```

<div class="aside">
<a href="https://storybook.js.org/addons/introduction/#1-decorators"><b>Decorators</b></a> 스토리에 임의의 wrapper를 제공하는 방법입니다. 이 경우는 기본 내보내기 decorator `key`를 이용해 렌더링된 컴포넌트 주변에 `padding`을 추가합니다. 또한 “providers”에서 스토리들을 랩핑하는데 사용될 수 있습니다. –i.e. React context을 구성하는 라이브러리 컴포넌트이다.
</div>

`taskData`는 우리가 만들고 내보낸 `Task.stories.js` 파일에서 `Task`의 모양을 제공합니다. 유사하게, `actionsData`는 `Task` 컴포넌트에서 생각하는 액션(mocked callbacks)을 정의하며, `TaskList`도 필요로 합니다.

이제 Storybook에서 새로운 `TaskList` 스토리를 확인합시다.

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/inprogress-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

## 상태 구축하기

우리의 컴포넌트는 아직 미가공 상태이지만 이제 우리는 앞으로 진행할 스토리에 대한 아이디어가 있습니다. `.list-items` wrapper는 지나치도록 단순하다고 생각할 수 있습니다. 그렇습니다 - 대부분의 경우 우리는 wrapper를 추가하기 위해 새로운 컴포넌트는 만들지 않습니다. 그러나 `TaskList` 컴포넌트의 **실제 복잡성**은 `withPinnedTasks`, `loading`, 그리고 `empty` 의 케이스에서 드러납니다.

```javascript
// src/components/TaskList.js

import React from 'react';

import Task from './Task';

function TaskList({ loading, tasks, onPinTask, onArchiveTask }) {
  const events = {
    onPinTask,
    onArchiveTask,
  };

  const LoadingRow = (
    <div className="loading-item">
      <span className="glow-checkbox" />
      <span className="glow-text">
        <span>Loading</span> <span>cool</span> <span>state</span>
      </span>
    </div>
  );

  if (loading) {
    return (
      <div className="list-items">
        {LoadingRow}
        {LoadingRow}
        {LoadingRow}
        {LoadingRow}
        {LoadingRow}
        {LoadingRow}
      </div>
    );
  }

  if (tasks.length === 0) {
    return (
      <div className="list-items">
        <div className="wrapper-message">
          <span className="icon-check" />
          <div className="title-message">You have no tasks</div>
          <div className="subtitle-message">Sit back and relax</div>
        </div>
      </div>
    );
  }

  const tasksInOrder = [
    ...tasks.filter(t => t.state === 'TASK_PINNED'),
    ...tasks.filter(t => t.state !== 'TASK_PINNED'),
  ];

  return (
    <div className="list-items">
      {tasksInOrder.map(task => (
        <Task key={task.id} task={task} {...events} />
      ))}
    </div>
  );
}

export default TaskList;
```

추가 된 마크업은 다음 UI에서 드러납니다:

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/finished-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

리스트에서 고정 된 항목의 위치를 확인 하십시요. 우리는 고정 된 항목을 리스트의 맨 위에 랜더링함으로써 사용자들에게 우선 순위를 보여주고자 합니다.

## 데이터 요구 사항 및 속성

컴포넌트가 커짐에 따라, 입력 요구 사항도 커집니다. `TaskList`의 prop 요구사항을 정의 하십시요. `Task`는 자식 컴포넌트로, 올바른 모양으로 렌더링하기 위한 데이터를 제공해야합니다. 시간과 복잡함을 줄이기 위해, 일찍이 `Task`에 정의된 propTypes를 재사용 하십시요.

```javascript
// src/components/TaskList.js

import React from 'react';
import PropTypes from 'prop-types';

import Task from './Task';

function TaskList() {
  ...
}


TaskList.propTypes = {
  loading: PropTypes.bool,
  tasks: PropTypes.arrayOf(Task.propTypes.task).isRequired,
  onPinTask: PropTypes.func.isRequired,
  onArchiveTask: PropTypes.func.isRequired,
};

TaskList.defaultProps = {
  loading: false,
};

export default TaskList;
```

## 자동화된 테스트

이전 장에서는 Storyshots를 이용한 테스트 스토리를 스냅샷 하는 법을 배웠습니다. `Task`를 함께하면 테스트할 복잡성이 많지 않았습니다. 이후 `TaskList`는 복잡성의 또다른 계층을 추가하므로 자동화 테스트에 적합한 방식으로 특정 입력이 특정 출력으로 생성되는 검증하려 합니다. 이를 위해 테스트 렌더러와 [Jest](https://facebook.github.io/jest/)를 이용해 단위 테스트를 만듭니다.

![Jest logo](/intro-to-storybook/logo-jest.png)

### Jest를 사용한 단위 테스트

매뉴얼 비쥬얼 테스트와 스냅샷 테스(위 참조)와 결합된 Storybook 스토리는 UI 버그를 피하는데 도움이 됩니다. 만약 스토리가 광 범위한 컴포넌트의 사용케이스를 다루고, 사람이 스토리 변경사항을 확인하도록 툴을 사용한다 오류들은 훨씬 줄어들겁니다.

그러나, 때론 devil은 세부 사항에 있습니다. 이런 세부 사항들에 있어 명확하게 하는 테스트 프레임워는 필요로 합니다. 이는 단위 테스트를 가능케 합니다.

우리의 경우, 우리의 `TaskList`가 `tasks` prop에서 고정되지 않은 task들 **이전에** 고정 된 task을 랜더 하고자 원합니다. 비록 우리는 이 정확한 시나리오를 테스트할 (`WithPinnedTasks`) 스토리가 있지만, 컴포넌트가 이와 같은 작업을 **정지** 한다면 인간 리뷰어에게 모호해지며, 이것은 버그입니다. 확실히 평범한 눈에는 **“잘못됐어!”** 라고 외치지 않을겁니다.

그래서, 이 문제를 피하기 위해서, 우리는 Jest를 사용하여 스토리에서 DOM으로 렌더링하고 일부 DOM 쿼리 코드를 실행해 출력의 현저 특징을 검증 할 수 있습니다. 스토리 포맷에 대한 좋은 점은 테스트에서 간단히 스토리를 가져 올 수 있고 렌더링 할 수 있다는 것입니다!

`src/components/TaskList.test.js`라고 부를 테스트 파일을 만듭시다. 여기에서는 출력에 대한 역설을 만드는 테스트를 작성합시다.

```javascript
// src/components/TaskList.test.js

import React from 'react';
import ReactDOM from 'react-dom';
import { WithPinnedTasks } from './TaskList.stories';

it('renders pinned tasks at the start of the list', () => {
  const div = document.createElement('div');
  ReactDOM.render(<WithPinnedTasks />, div);

  // We expect the task titled "Task 6 (pinned)" to be rendered first, not at the end
  const lastTaskInput = div.querySelector('.list-item:nth-child(1) input[value="Task 6 (pinned)"]');
  expect(lastTaskInput).not.toBe(null);

  ReactDOM.unmountComponentAtNode(div);
});
```

![TaskList test runner](/intro-to-storybook/tasklist-testrunner.png)

우리는 `WithPinnedTasks` 스토리를 단위테스트에서 재사용 할 수 있었습니다; 이러한 방법으로 우리는 존재하는 리소스(컴포넌트의 흥미로운 요소들을 나타내는 예제들)를 여러 방법으로 계속 사용 가능합니다.

아직 이 테스트는 매우 취약합니다. 프로젝트의 성숙함에 따라, 그리고 `Task` 변경의 정확한 구현 --아마도 다른 classname 또는 `input` 보다는 `textarea`을 사용한다면 --테스트는 실패하고 업데이트 해야합니다. 이것은 필연적인 문제가 아닙니다, 그러나 UI에 대한 단위 테스트를 자유로이 사용하는것에 대해 주의해야 한다고 표시 합니다. 그것들은 유지관리 하기 쉽지 않습니다. 대신에 가능한 경우 비쥬얼, 스냅샷, 비쥬얼 회귀 (참조 [testing chapter](/test/)) 테스트에 의존 하십시요.
