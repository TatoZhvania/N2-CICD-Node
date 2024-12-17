### `production.yml` ფაილის ახსნა

ეს არის GitHub Actions ვორქფლოუ, რომელიც ავტომატურად აგებს, ტესტავს და აწვდის Production გარემოში Node.js აპლიკაციას Vercel-ზე.

---

### სტრუქტურის დეტალური ახსნა

### 1. Trigger

ვორქფლოუ ირთვება `main` ბრენჩზე ცვლილებების დაპუშვისას:

```yaml
on:
  push:
    branches:
      - main
```

ეს ნიშნავს, რომ ვორქფლოუ ავტომატურად გაეშვება მხოლოდ მაშინ, როცა `main` ბრენჩზე აიტვირთება ახალი ცვლილებები.

---

### **2. Environment Variables**

Vercel-ის კონფიგურაციისთვის საჭირო პარამეტრები (ორგანიზაციისა და პროექტის ID) მოდის GitHub Secrets-დან:

```yaml
env:
  VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
  VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
```

ეს უზრუნველყოფს სენსიტიური ინფორმაციის დაცვას და უსაფრთხო გამოყენებას.

---

### **3**. Jobs

ვორქფლოუ იყოფა სამ ძირითად სამუშაოდ: Build, Test, და Deploy.

---

### Job 1 - Build (დაბილდვა)

```yaml
jobs:
  Build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install Dependencies
        run: npm ci

      - name: Build Application
        run: npm run build
```

`Build`: აგებს აპლიკაციას.

ნაბიჯები:

კოდის გამოტანა: `actions/checkout@v3` იღებს პროექტის კოდს GitHub-დან.

Node.js ვერსია 18-ის დაყენება.

`npm ci` გამოიყენება ზუსტი დამოკიდებულებების დასაყენებლად.

`npm run build` აწყობს პროექტის საბოლოო არტეფაქტებს.

---

### **Job 2 - Test (ტესტირება)**

```yaml
yaml
Copy code
  Test:
    runs-on: ubuntu-latest
    needs: Build
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install Dependencies
        run: npm ci

      - name: Run Tests
        run: npm test

```

**`Test`**: ტესტავს აპლიკაციას, მხოლოდ Build ჯობის წარმატებით დასრულების შემდეგ.

ნაბიჯები:

იგივე პროცესები, რაც Build ჯობში (კოდის გამოტანა, Node.js დაყენება, დამოკიდებულებების ინსტალაცია).

ტესტების გაშვება: `npm test` ამოწმებს პროექტს.

---

### **Job 3 - Deploy ( განთავსება)**

```yaml
  Deploy:
    runs-on: ubuntu-latest
    needs: Test
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Install Vercel CLI
        run: npm install --global vercel

      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build Project Artifacts
        run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy Project Artifacts
        run: vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}

```

`Deploy`: აპლიკაცია განთავსდება Vercel-ის Production გარემოში, თუ ტესტები წარმატებით დასრულდება (`needs: Test`).

ნაბიჯები:

Vercel CLI-ის დაყენება: აპლიკაციის გადასატანად ინსტალირდება Vercel CLI.

Production გარემოს კონფიგურაციის გამოტანა:ეს უკავშირდება Vercel-ს და გამოაქვს Production კონფიგურაცია.

```bash
vercel pull --yes --environment=production
```

აპლიკაციის აგება: `vercel build` ამზადებს პროექტის არტეფაქტებს.

აპლიკაციის განთავსება: `vercel deploy` აწვდის აპლიკაციას Production გარემოში.

### `preview.yml` ფაილის ახსნა

ეს არის GitHub Actions ვორქფლოუ, რომელიც Preview გარემოში ავტომატურად ტესტავს და აწვდის Node.js აპლიკაციას Vercel-ზე. ვორქფლოუ ირთვება იმ შემთხვევაში, თუ ცვლილებები დაპუშულია ნებისმიერ ბრენჩზე, გარდა `main`-ისა.

---

სინტაქსი productionსა და preview ფაილებს მსგავსი აქვთ გარდა ერთისა. 

ვორქფლოუ ირთვება `main` ბრენჩის გარდა ყველა სხვა ბრენჩზე:

```yaml
on:
  push:
    branches-ignore:
      - main
```

ეს უზრუნველყოფს, რომ Preview Deployment განხორციელდეს მხოლოდ `main`-ის გარეთ არსებული ბრენჩებისთვის.
