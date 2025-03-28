

# next.js15

##  `\`默认主页面


![[Pasted image 20241205092711.png]]

page.tsx、layout.tsx 是这个工程的入口页面，指定了首页和要加载的页面。


[layout.tsx](https://nextjs.org/docs/app/building-your-application/routing/layouts-and-templates#root-layout-required)：布局组件，分为根布局、嵌套布局。
根布局(必须)：app/layout.tsx
![[Pasted image 20241205100707.png]]
该文件相当于真正定义主页面，
![[Pasted image 20241205100840.png]]




**在 Next.js 中，`RootLayout` 组件会自动包裹应用中的所有页面组件，包括 `Home` 组件。这是 Next.js 的默认行为，通过在 `app` 目录下定义 `layout.tsx` 文件来实现。当你访问根路径 `/` 时，Next.js 会自动加载 `app/page.tsx` 文件中的 `Home` 组件，并将其作为 `children` 传递给 `RootLayout` 组件。**














