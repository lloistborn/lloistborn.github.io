## The Initial Approach: Direct Component State

Like many developers, I started with what seemed like a straightforward task: displaying dynamic page titles based on the selected sidebar item in my Next.js application. My first attempt was refreshingly simple - just add a title to each page component:

```jsx
export default function NotificationsPage() {
    return (
        <h1>Notifications</h1>
    )
}
```

## DRY Principles
As the application grew, I realized I was repeating these titles in multiple places:
- In the sidebar navigation
- In the page headers
- Potentially in breadcrumbs

This violated the DRY (Don't Repeat Yourself) principle. I needed a better solution.

## The Navigation-Driven Approach
I decided to make the navigation items the single source of truth. I enhanced my navigation items in `AppLayout.tsx` to include a `title` property:

```jsx
const navigationItems = [
  { 
    name: 'Goal', 
    path: '/', 
    icon: '/goal.svg', 
    isVisible: true,
    pageTitle: 'Dashboard'
  },
  { 
    name: 'Notifications', 
    path: '/notifications', 
    icon: '/notification.svg', 
    isVisible: user ? true : false,
    pageTitle: 'Notifications'
  }
  // ... more items
]
```

## The Context Conundrum
To make these titles available throughout the app, I created a context:

```jsx
const PageContext = createContext<PageContextType>({ currentPageTitle: undefined })

export function PageProvider({ children, currentPageTitle }: {
  children: React.ReactNode,
  currentPageTitle?: string 
}) {
  return (
    <PageContext.Provider value={{ currentPageTitle }}>
      {children}
    </PageContext.Provider>
  )
}
```

## The Server-Side Surprise
But then I hit a wall! When I tried to use the context in my page components:
```jsx
// This caused an error!
const { currentPageTitle } = usePageContext()
```
I got the dreaded error:
> Attempted to call usePageContext() from the server but usePageContext is on the client.

## The Final Solution: Client Components to the Rescue
The solution was to create a dedicated client component for the `pagetitle.tsx`:
```jsx
'use client'

import { usePageContext } from '@/context/PageContext'

export default function PageTitle() {
  const { currentPageTitle } = usePageContext()
  
  return (
    <h1 className="text-2xl font-semibold text-gray-900">
      {currentPageTitle}
    </h1>
  )
}
```

And use it in our server-side page components `page.tsx`:
```jsx
export default function NotificationsPage() {
    return (
        <div>
            <PageTitle />
            {/* rest of the page content */}
        </div>
    )
}
```

## Lessons Learned
- Single Source of Truth: Keeping navigation and page metadata in one place makes maintenance easier

- Server vs Client Components: Understanding Next.js's server/client boundary is crucial

- Component Separation: Sometimes creating a dedicated component for a small feature is the right approach

- Context Usage: While contexts are powerful, they need to be used carefully in Next.js's server-component world

- Conclusion
What started as a simple task turned into a journey through Next.js's architecture and React best practices. The final solution is clean, maintainable, and properly handles the server/client component boundary.
Remember: in web development, the simplest-looking features often lead to the most interesting learning experiences!
