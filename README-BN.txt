ARIF ALL DOCUMENT — শুরু করার নিয়ম (বাংলায়)
==============================================

এই প্রজেক্টে আছে:
- একাধিক ব্যবহারকারীর Email + Password account
- Login / Sign up / Password reset
- ব্যক্তিগত Dashboard
- PDF, ছবি, Office, ZIP, audio/video ফাইল upload
- নিজের ফাইল download ও delete
- অন্য ব্যবহারকারীর ফাইল থেকে আলাদা রাখার জন্য Supabase RLS policy

গুরুত্বপূর্ণ:
GitHub repository-তে শুধু website-এর code রাখবেন। ব্যবহারকারীর আসল নথি GitHub-এ রাখবেন না।
Supabase Storage bucket private থাকবে। Service role key কখনো app.js বা GitHub-এ দেবেন না।
শুধু Project URL এবং anon/public key ব্যবহার করবেন।

ধাপ ১ — Supabase project তৈরি
1. https://supabase.com এ যান এবং account খুলুন।
2. New project চাপুন।
3. Project name দিন: arif-all-document
4. Database password শক্তিশালী দিন এবং নিরাপদে সংরক্ষণ করুন।
5. Region হিসেবে কাছাকাছি উপলভ্য region বেছে নিন।
6. Project তৈরি হওয়া পর্যন্ত অপেক্ষা করুন।

ধাপ ২ — Database ও Storage নিরাপত্তা তৈরি
1. Supabase Dashboard-এ SQL Editor খুলুন।
2. এই ZIP-এর supabase-schema.sql ফাইল খুলে সম্পূর্ণ SQL কপি করুন।
3. SQL Editor-এ paste করে Run চাপুন।
4. Storage > Buckets-এ user-documents bucket-টি Private আছে কি না দেখুন।

ধাপ ৩ — Email sign-up সেটিং
1. Authentication > Providers > Email চালু রাখুন।
2. Authentication > URL Configuration-এ আপনার deployed website URL পরে যোগ করুন।
3. Email confirmation চালু থাকলে signup-এর পর inbox থেকে email confirm করতে হবে।
4. Production-এ নিজের domain এবং সঠিক redirect URL ব্যবহার করুন।

ধাপ ৪ — Supabase key বসান
1. Project Settings > API (বা Connect/API Keys) খুলুন।
2. Project URL কপি করুন।
3. anon/public key কপি করুন। নতুন dashboard-এ publishable key থাকলে সেটিও client-side ব্যবহারের জন্য।
4. app.js ফাইলে এই দুই লাইনে বসান:
   const SUPABASE_URL = "PASTE_YOUR_SUPABASE_PROJECT_URL";
   const SUPABASE_ANON_KEY = "PASTE_YOUR_SUPABASE_ANON_KEY";
5. কখনো service_role / secret key এখানে বসাবেন না।

ধাপ ৫ — GitHub-এ upload
1. আপনার GitHub repository Arif_All_Document খুলুন।
2. Add file (+) > Upload files নির্বাচন করুন।
3. index.html, style.css, app.js এবং supabase-schema.sql upload করুন।
4. README-BN.txt-ও রাখতে পারেন।
5. Commit changes করুন।

ধাপ ৬ — Website চালু
সহজ উপায়: https://vercel.com এ GitHub দিয়ে sign in করুন।
1. Add New > Project নির্বাচন করুন।
2. Arif_All_Document repository import করুন।
3. Framework preset: Other রাখুন।
4. Deploy চাপুন।
5. তৈরি হওয়া website URL Supabase Authentication URL Configuration-এ যোগ করুন।

পরীক্ষা
- প্রথমে একটি account তৈরি করুন এবং email confirmation সম্পন্ন করুন।
- Login করে একটি নিরীহ test PDF upload করুন।
- Logout করে দ্বিতীয় account খুলুন।
- দ্বিতীয় account-এ প্রথম account-এর ফাইল দেখা যায় না—এটি পরীক্ষা করুন।
- তারপর দ্বিতীয় account থেকে upload/download/delete পরীক্ষা করুন।

নিরাপত্তা ও সীমাবদ্ধতা
- এই starter project-এ প্রতি ফাইলের সীমা 50 MB।
- Supabase plan-এর storage/traffic limit এবং খরচ দেখে নিন।
- Supabase Storage-এ ফাইল private; download-এর জন্য স্বল্পমেয়াদি signed URL তৈরি হয়।
- গুরুত্বপূর্ণ আসল নথি রাখার আগে security পরীক্ষা ও আলাদা backup রাখুন।
- Aadhaar, PAN, bank papers-এর মতো sensitive documents ব্যবহারের আগে access policy, account recovery, MFA এবং privacy/security review সম্পন্ন করুন।
- এই প্রাথমিক সংস্করণে admin panel নেই; প্রতিটি user নিজের ফাইলই নিয়ন্ত্রণ করে।
