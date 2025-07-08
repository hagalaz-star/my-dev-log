네, 알겠습니다! "그대로 복사해서 붙여넣으면 보기 안 좋다"는 말씀이 정확히 맞습니다. README는 프로젝트의 얼굴이니, 깔끔하고 보기 좋게 만드는 것이 매우 중요하죠.

시각적인 부분과 가독성을 대폭 개선하여, 다른 개발자들이 봤을 때 "이 프로젝트, 정말 체계적으로 만들었다!"라는 인상을 줄 수 있도록 Supabase 초기 설정 가이드 부분을 완전히 새롭게 다듬어 드리겠습니다.

아래 내용을 그대로 복사해서 README.md 파일의 해당 부분을 교체해 보세요.

## ⚙️ Supabase 초기 설정 가이드

이 프로젝트는 사용자 인증과 데이터베이스 기능을 위해 Supabase를 사용합니다. Next.js App Router 환경에서 Supabase를 안전하고 효율적으로 사용하기 위한 초기 설정 과정은 다음과 같습니다.

1단계: 패키지 설치
Supabase와 상호작용하고, 서버사이드 렌더링 환경에서 인증을 관리하기 위한 라이브러리를 설치합니다.

Bash

npm install @supabase/supabase-js @supabase/ssr
@supabase/supabase-js: Supabase와 통신하는 핵심 클라이언트 라이브러리입니다.

@supabase/ssr: Next.js와 같은 SSR 환경에서 안전하게 인증 쿠키를 관리하도록 돕는 헬퍼 라이브러리입니다.

2단계: 환경 변수 설정
프로젝트의 민감한 정보(API Key)를 보호하기 위해, 프로젝트 루트에 .env.local 파일을 생성하고 아래와 같이 키를 추가합니다. 이 파일은 Git에 의해 추적되지 않아 키 유출을 방지합니다.

NEXT_PUBLIC_SUPABASE_URL=<your_supabase_project_url>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your_supabase_anon_key>
💡 왜 .env.local을 사용하나요?
이 파일은 Git에 의해 추적되지 않으므로(무시되므로), GitHub 같은 공개된 공간에 프로젝트의 중요한 비밀 키가 유출되는 것을 원천적으로 방지합니다.

3단계: Supabase 클라이언트 생성 (역할 분담)
Next.js 환경에서는 코드가 실행되는 위치(서버/브라우저)가 다르므로, 각 환경에 맞는 별도의 클라이언트를 만들어 역할을 분리합니다. 이는 보안과 성능을 위한 핵심적인 구조입니다.

서버용 (금고 관리자 🏦)

클라이언트용 (로비 안내원 💁)

utils/supabase/server.ts

utils/supabase/client.ts

서버 액션, API 라우트처럼 서버에서만 실행됩니다. DB에 직접 접근하는 모든 강력한 권한을 가집니다.

사용자의 웹 브라우저에서만 실행됩니다. 사용자의 클릭에 반응하여 제한된 업무만 수행합니다.

```typescript

import { createServerClient } from '@supabase/ssr'

import { cookies } from 'next/headers'


Sheets로 내보내기
export async function createClient() {
const cookieStore = await cookies()
return createServerClient(
process.env.NEXT_PUBLIC_SUPABASE_URL!,
process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
{
cookies: {
getAll() {
return cookieStore.getAll()
},
setAll(cookiesToSet) {
try {
cookiesToSet.forEach(
({ name, value, options }) =>
cookieStore.set(name, value, options)
)
} catch {
// 서버 컴포넌트에서 쿠키를 설정하려고 할 때 에러가 발생할 수 있습니다.
}
},
},
}
)
}
|typescript
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
return createBrowserClient(
process.env.NEXT_PUBLIC_SUPABASE_URL!,
process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)
}
```
