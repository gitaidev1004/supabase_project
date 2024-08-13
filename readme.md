# 1. Supabase 프로젝트 기본 실습

Supabase는 PostgreSQL 데이터베이스를 백엔드로 사용하고 서버리스 함수, 인증, 실시간 데이터베이스 기능 등을 제공하는 오픈 소스 백엔드 플랫폼입니다. 

<br>

## 1-1. Supabase 프로젝트 설정

### 1-1-1. Supabase 계정 생성 및 프로젝트 만들기

1. Supabase 웹사이트(https://supabase.com)로 이동하여 계정을 생성합니다.
2. 로그인 후, "New Project" 버튼을 클릭하여 새로운 프로젝트를 만듭니다.
3. 프로젝트의 이름을 입력하고, 데이터베이스 비밀번호를 설정합니다.
4. Region과 Pricing Plan을 선택하고 "Create New Project" 버튼을 클릭합니다.

<br><br>

### 1-1-2. Supabase 설정

프로젝트가 생성되면, Supabase 대시보드에 접속하게 됩니다.
"Settings" -> "API" 섹션에서 Project URL과 anon key를 확인합니다. 이 키는 클라이언트 애플리케이션에서 Supabase API와 통신하는 데 필요합니다.
Database 섹션에서 데이터베이스 URL과 정보도 확인할 수 있습니다.

<br><br><br>

## 1-2. 애플리케이션 개발

이제 Supabase와 연동된 간단한 풀스택 애플리케이션을 생성해보겠습니다. 이 예제에서는 Next.js와 Supabase 클라이언트 라이브러리를 사용합니다.

<br>

### 1-2-1. 프로젝트 초기화

프로젝트 디렉토리를 생성하고, Next.js 프로젝트를 초기화합니다:

```bash
mkdir supabase-next-app
cd supabase-next-app
npx create-next-app@latest .
```

<br>

Supabase 클라이언트 라이브러리를 설치합니다:

```bash
npm install @supabase/supabase-js
```

<br><br>

### 1-2-2. Supabase 클라이언트 설정

Supabase와 통신하기 위해 클라이언트를 설정합니다.

lib/supabaseClient.js 파일을 생성하고 다음 코드를 추가합니다:

```javascript
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

<br>

.env.local 파일을 생성하고, Supabase 프로젝트의 API URL과 anon key를 설정합니다:

```makefile
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

<br><br>

### 1-2-3. 인증 구현

Next.js와 Supabase를 사용하여 간단한 인증 시스템을 구축할 수 있습니다.

**pages/_app.js** 파일에 Supabase 클라이언트를 추가하여 전역 상태로 관리합니다:

```javascript
import { useEffect } from 'react'
import { supabase } from '../lib/supabaseClient'

function MyApp({ Component, pageProps }) {
  useEffect(() => {
    const { data: authListener } = supabase.auth.onAuthStateChange((event, session) => {
      if (session) {
        console.log('User signed in:', session.user)
      } else {
        console.log('User signed out')
      }
    })

    return () => {
      authListener.unsubscribe()
    }
  }, [])

  return <Component {...pageProps} />
}

export default MyApp
```

<br>

**pages/login.js** 파일을 생성하고, 사용자 로그인 페이지를 작성합니다:

```javascript
import { useState } from 'react'
import { supabase } from '../lib/supabaseClient'

export default function Login() {
  const [email, setEmail] = useState('')

  const handleLogin = async () => {
    const { error } = await supabase.auth.signInWithOtp({ email })
    if (error) console.error('Error logging in:', error.message)
    else alert('Check your email for the login link!')
  }

  return (
    <div>
      <h1>Login</h1>
      <input
        type="email"
        placeholder="Enter your email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <button onClick={handleLogin}>Login</button>
    </div>
  )
}
```

<br><br>

### 1-2-4. 데이터베이스와 연동

데이터를 가져오고 보여주는 간단한 예제를 작성합니다.

Supabase 대시보드에서 "Table Editor"를 사용하여 messages 테이블을 생성합니다. 이 테이블은 다음과 같은 컬럼을 포함합니다:

```
id (int, primary key, auto increment)
content (text)
created_at (timestamp with time zone, default: now())
```

pages/index.js 파일을 수정하여 메시지를 가져오고 표시하는 코드를 작성합니다:

```javascript
import { useEffect, useState } from 'react'
import { supabase } from '../lib/supabaseClient'

export default function Home() {
  const [messages, setMessages] = useState([])

  useEffect(() => {
    const fetchMessages = async () => {
      let { data, error } = await supabase
        .from('messages')
        .select('*')
      if (error) console.error('Error fetching messages:', error.message)
      else setMessages(data)
    }

    fetchMessages()
  }, [])

  return (
    <div>
      <h1>Messages</h1>
      <ul>
        {messages.map((message) => (
          <li key={message.id}>{message.content}</li>
        ))}
      </ul>
    </div>
  )
}
```

<br><br><br>

## 1-3. 서버리스 함수 구현

Supabase는 서버리스 함수 기능을 제공합니다. 이를 통해 클라이언트에서 사용할 수 있는 서버리스 API를 쉽게 작성할 수 있습니다.

Supabase 대시보드에서 "Functions" 섹션으로 이동하여 새 함수를 생성합니다.

hello-world라는 이름의 함수를 만들고 다음 코드를 작성합니다:

```javascript
export default async function handler(req, res) {
  res.status(200).json({ message: 'Hello World!' })
}
```

이 함수를 배포하고, 함수 URL을 통해 호출할 수 있습니다.

## 1-4. 테스트 및 배포

### 1-4-1. 로컬 테스트

로컬에서 Next.js 앱을 실행하여 테스트합니다:

```bash
npm run dev
```

브라우저에서 http://localhost:3000에 접속하여 애플리케이션이 제대로 동작하는지 확인합니다.

<br><br>

### 1-4-2. 배포

Next.js 애플리케이션을 배포하기 위해 Vercel을 사용할 수 있습니다.

Vercel에 로그인하고 새로운 프로젝트를 생성합니다.
GitHub 리포지토리를 연결하고 배포를 시작합니다.
.env.local 파일의 환경 변수를 Vercel 설정에서 구성합니다.

<br><br><br>

## 1-5. 프로젝트 파일 구조

프로젝트 파일 트리 구조는 다음과 같습니다:

```lua
supabase-next-app/
│
├── lib/
│   └── supabaseClient.js
│
├── pages/
│   ├── _app.js
│   ├── index.js
│   └── login.js
│
├── .env.local
├── package.json
└── next.config.js
```
