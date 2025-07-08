⚙️ Supabase 초기 설정
이 프로젝트는 사용자 인증과 데이터베이스 기능을 위해 Supabase를 사용합니다. Next.js App Router 환경에서 Supabase를 안전하고 효율적으로 사용하기 위한 초기 설정 과정은 다음과 같습니다.

1단계: 패키지 설치
Supabase와 상호작용하고, 서버사이드 렌더링 환경에서 인증을 관리하기 위한 라이브러리를 설치합니다.

Bash

npm install @supabase/supabase-js @supabase/ssr
@supabase/supabase-js: Supabase와 통신하는 핵심 클라이언트 라이브러리입니다.

@supabase/ssr: Next.js와 같은 SSR 환경에서 안전하게 인증 쿠키를 관리하도록 돕는 헬퍼 라이브러리입니다.

2단계: 환경 변수 설정
프로젝트의 민감한 정보(API Key)를 보호하기 위해, 프로젝트 루트에 .env.local 파일을 생성하고 아래와 같이 키를 추가합니다.

NEXT_PUBLIC_SUPABASE_URL=<your_supabase_project_url>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your_supabase_anon_key>
💡 왜 .env.local을 사용하나요?

이 파일은 Git에 의해 추적되지 않으므로(무시되므로), GitHub 같은 공개된 공간에 프로젝트의 중요한 비밀 키가 유출되는 것을 원천적으로 방지합니다.

3단계: Supabase 클라이언트 생성 (역할 분담)
Next.js 환경에서는 코드가 실행되는 위치(서버/브라우저)가 다르므로, 각 환경에 맞는 별도의 클라이언트를 만들어 역할을 분리합니다. 이는 보안과 성능을 위한 핵심적인 구조입니다.

🖥️ 서버용 클라이언트 (금고 관리자)
서버 액션, API 라우트처럼 서버에서만 실행됩니다. DB에 직접 접근하는 모든 강력한 권한을 가집니다.

📁 파일 위치: src/utils/supabase/server.ts

TypeScript

import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { cookies } from 'next/headers'

export function createClient() {
const cookieStore = cookies()

return createServerClient(
process.env.NEXT_PUBLIC_SUPABASE_URL!,
process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
{
cookies: {
get(name: string) {
return cookieStore.get(name)?.value
},
set(name: string, value: string, options: CookieOptions) {
try {
cookieStore.set({ name, value, ...options })
} catch (error) {
// The `set` method was called from a Server Component.
}
},
remove(name: string, options: CookieOptions) {
try {
cookieStore.set({ name, value: '', ...options })
} catch (error) {
// The `delete` method was called from a Server Component.
}
},
},
}
)
}
☁️ 클라이언트용 클라이언트 (로비 안내원)
사용자의 웹 브라우저에서만 실행됩니다. 사용자의 클릭에 반응하여 제한된 업무만 수행하며, 서버의 민감한 정보에는 접근할 수 없습니다.

📁 파일 위치: src/utils/supabase/client.ts

TypeScript

import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
return createBrowserClient(
process.env.NEXT_PUBLIC_SUPABASE_URL!,
process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)
}
