
flowchart TB
    A["React Frontend<br/>TypeScript · Vite"]
    B["TanStack Query<br/>Zustand · Tailwind CSS"]
    C["Express REST API<br/>Node.js · TypeScript · Zod"]
    D["Prisma ORM"]
    E[("PostgreSQL<br/>Neon")]
    F["Firebase Authentication<br/>Firebase Admin"]
    G[("Cloudflare R2<br/>Object Storage")]
    H["Vercel<br/>Frontend Hosting"]
    I["Render<br/>Backend Hosting"]

    A --> B
    B --> C
    C --> D
    D --> E
    A --> F
    C --> F
    C --> G
    A -. deployed on .-> H
    C -. deployed on .-> I
