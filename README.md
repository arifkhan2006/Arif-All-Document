# Arif-All-Document
All Documents, PDF, Images and Important Files

<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Arif All Document</title>
<link rel="stylesheet" href="style.css">
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
</head>
<body>

<header>
  <h2>📁 ARIF ALL DOCUMENT</h2>
  <p>Your Personal Document Manager</p>
</header>

<section id="auth">
  <h2>Welcome Back</h2>
  <input id="email" type="email" placeholder="Email">
  <input id="password" type="password" placeholder="Password">
  <button onclick="login()">Login</button>
  <button onclick="signup()">Create Account</button>
  <p id="message"></p>
</section>

<main id="dashboard" hidden>
  <h2 id="welcome">My Dashboard</h2>
  <button onclick="logout()">Logout</button>

  <div class="stats">
    <div>Total Files: <b id="total">0</b></div>
    <div>Storage Used: <b id="storage">0 MB</b></div>
  </div>

  <h3>Upload Documents</h3>
  <input type="file" id="files" multiple>
  <button onclick="uploadFiles()">Upload Files</button>

  <h3>Create Folder</h3>
  <input id="folderName" placeholder="Folder Name">
  <button onclick="createFolder()">Create Folder</button>
  <div id="folders"></div>

  <h3>Search Documents</h3>
  <input id="search" placeholder="Search file..." oninput="showFiles()">

  <h3>My Documents</h3>
  <button onclick="selectAll()">Select All</button>
  <button onclick="bulkDelete()">Delete Selected</button>
  <div id="fileList"></div>

  <section id="admin" hidden>
    <h2>Admin Panel</h2>
    <p>Total Users: <b id="users">0</b></p>
  </section>

  <h3>Account Settings</h3>
  <input id="displayName" placeholder="Your Name">
  <input type="file" id="avatar" accept="image/*">
  <button onclick="saveProfile()">Save Profile</button>
  <img id="profilePhoto" width="70">
</main>

<div id="preview" class="preview" hidden>
  <button onclick="closePreview()">Close</button>
  <div id="previewContent"></div>
</div>

<script src="app.js"></script>
</body>
</html>

* {
  box-sizing: border-box;
}
body {
  margin: 0;
  padding: 20px;
  background: #f1f5fb;
  font-family: Arial, sans-serif;
  color: #172033;
}
header {
  background: #173d78;
  color: white;
  padding: 22px;
  border-radius: 15px;
  text-align: center;
  margin-bottom: 25px;
}
section, main {
  max-width: 850px;
  margin: auto;
}
#auth, .stats > div {
  background: white;
  padding: 22px;
  border-radius: 14px;
  box-shadow: 0 3px 15px #0001;
}
input {
  display: block;
  width: 100%;
  padding: 13px;
  margin: 12px 0;
  border: 1px solid #ccd5e2;
  border-radius: 9px;
}
button {
  background: #2454a6;
  color: white;
  border: 0;
  padding: 11px 17px;
  margin: 5px 3px;
  border-radius: 8px;
  cursor: pointer;
}
button:hover {
  background: #163d7d;
}
.stats {
  display: flex;
  gap: 12px;
  margin: 20px 0;
}
.stats > div {
  flex: 1;
}
#fileList, #folders {
  margin: 15px 0;
}
.file {
  background: white;
  padding: 13px;
  border-radius: 10px;
  margin: 9px 0;
  overflow-wrap: anywhere;
}
.preview {
  position: fixed;
  inset: 5%;
  background: white;
  padding: 20px;
  overflow: auto;
  border-radius: 15px;
  box-shadow: 0 0 0 100vmax #0009;
}
.preview img {
  max-width: 100%;
}
.preview iframe {
  width: 100%;
  height: 75vh;
}
[hidden] {
  display: none !important;
}

const SUPABASE_URL = "YOUR_PROJECT_URL";
const SUPABASE_KEY = "YOUR_ANON_PUBLIC_KEY";

const db = window.supabase.createClient(
  SUPABASE_URL,
  SUPABASE_KEY
);

let currentUser = null;
let documents = [];
let selected = new Set();

async function signup() {
  const {data,error} = await db.auth.signUp({
    email: email.value,
    password: password.value
  });
  message.textContent = error
    ? error.message
    : "Account created. Email confirm করে Login করুন।";
}

async function login() {
  const {data,error} = await db.auth.signInWithPassword({
    email: email.value,
    password: password.value
  });
  if(error) return message.textContent = error.message;
  currentUser = data.user;
  await startApp();
}

async function startApp() {
  auth.hidden = true;
  dashboard.hidden = false;
  welcome.textContent = "Welcome " + currentUser.email;
  await loadFiles();
  await loadFolders();
  await checkAdmin();
}

async function logout() {
  await db.auth.signOut();
  location.reload();
}

async function uploadFiles() {
  const selectedFiles = [...files.files];
  for(const file of selectedFiles) {
    if(file.size > 50 * 1024 * 1024) {
      alert(file.name + " 50 MB-এর বেশি");
      continue;
    }

    const path = currentUser.id + "/" +
      crypto.randomUUID() + "-" + file.name.replaceAll("/", "_");

    const {error} = await db.storage
      .from("user-documents")
      .upload(path,file);

    if(error) {
      alert(error.message);
      continue;
    }

    const result = await db.from("documents").insert({
      owner_id: currentUser.id,
      storage_path: path,
      file_name: file.name,
      mime_type: file.type || "application/octet-stream",
      size_bytes: file.size
    });

    if(result.error) alert(result.error.message);
  }

  files.value = "";
  await loadFiles();
}

async function loadFiles() {
  const {data,error} = await db.from("documents")
    .select("*").order("created_at",{ascending:false});

  if(error) return alert(error.message);
  documents = data || [];
  total.textContent = documents.length;

  const bytes = documents.reduce(
    (sum,f)=>sum + Number(f.size_bytes),0
  );
  storage.textContent = (bytes/1048576).toFixed(2)+" MB";
  showFiles();
}

function showFiles() {
  const query = search.value.toLowerCase();
  fileList.innerHTML = "";

  documents.filter(f=>f.file_name.toLowerCase().includes(query))
  .forEach(f=>{
    const div = document.createElement("div");
    div.className = "file";

    const checkbox = document.createElement("input");
    checkbox.type = "checkbox";
    checkbox.onchange = () => {
      checkbox.checked ? selected.add(f.id) : selected.delete(f.id);
    };

    const name = document.createElement("b");
    name.textContent = f.file_name;

    const previewBtn = document.createElement("button");
    previewBtn.textContent = "Preview";
    previewBtn.onclick = ()=>previewFile(f);

    const downloadBtn = document.createElement("button");
    downloadBtn.textContent = "Download";
    downloadBtn.onclick = ()=>downloadFile(f);

    const renameBtn = document.createElement("button");
    renameBtn.textContent = "Rename";
    renameBtn.onclick = ()=>renameFile(f);

    const deleteBtn = document.createElement("button");
    deleteBtn.textContent = "Delete";
    deleteBtn.onclick = ()=>deleteFile(f);

    div.append(checkbox,name,previewBtn,downloadBtn,renameBtn,deleteBtn);
    fileList.append(div);
  });
}

async function downloadFile(f) {
  const {data,error} = await db.storage
    .from("user-documents")
    .createSignedUrl(f.storage_path,120);

  if(error) return alert(error.message);

  const a = document.createElement("a");
  a.href = data.signedUrl;
  a.download = f.file_name;
  a.click();
}

async function previewFile(f) {
  const {data,error} = await db.storage
    .from("user-documents")
    .createSignedUrl(f.storage_path,300);

  if(error) return alert(error.message);

  previewContent.replaceChildren();

  if(f.mime_type.startsWith("image/")) {
    const img = document.createElement("img");
    img.src = data.signedUrl;
    previewContent.append(img);
  } else if(f.mime_type === "application/pdf") {
    const iframe = document.createElement("iframe");
    iframe.src = data.signedUrl;
    previewContent.append(iframe);
  } else {
    alert("এই ফাইলের Preview নেই। Download করুন।");
    return;
  }

  preview.hidden = false;
}

function closePreview() {
  preview.hidden = true;
}

async function renameFile(f) {
  const name = prompt("নতুন ফাইলের নাম:",f.file_name);
  if(!name || !name.trim()) return;

  const {error} = await db.from("documents")
    .update({file_name:name.trim()})
    .eq("id",f.id);

  if(error) return alert(error.message);
  await loadFiles();
}

async function deleteFile(f) {
  if(!confirm("ফাইল Delete করবেন?")) return;
  const {error} = await db.storage.from("user-documents")
    .remove([f.storage_path]);

  if(error) return alert(error.message);

  await db.from("documents").delete().eq("id",f.id);
  selected.delete(f.id);
  await loadFiles();
}

async function bulkDelete() {
  const list = documents.filter(f=>selected.has(f.id));
  if(!list.length || !confirm("Selected files Delete করবেন?")) return;

  for(const f of list) {
    const {error} = await db.storage.from("user-documents")
      .remove([f.storage_path]);
    if(error) {
      alert(error.message);
      continue;
    }
    await db.from("documents").delete().eq("id",f.id);
  }

  selected.clear();
  await loadFiles();
}

function selectAll() {
  documents.forEach(f=>selected.add(f.id));
  showFiles();
}

async function createFolder() {
  const name = folderName.value.trim();
  if(!name) return;

  const {error} = await db.from("folders").insert({
    owner_id:currentUser.id,
    name
  });

  if(error) return alert(error.message);
  folderName.value = "";
  loadFolders();
}

async function loadFolders() {
  const {data,error} = await db.from("folders")
    .select("*").order("name");

  if(error) return alert(error.message);

  folders.innerHTML = "";
  (data||[]).forEach(f=>{
    const p = document.createElement("p");
    p.textContent = "📁 " + f.name;
    folders.append(p);
  });
}

async function saveProfile() {
  let avatarPath = null;
  const photo = avatar.files[0];

  if(photo) {
    if(photo.size > 3*1024*1024)
      return alert("ছবি 3 MB-এর মধ্যে রাখুন");

    avatarPath = currentUser.id + "/" +
      crypto.randomUUID() + "-" + photo.name;

    const {error} = await db.storage.from("user-avatars")
      .upload(avatarPath,photo);

    if(error) return alert(error.message);
  }

  const {error} = await db.from("profiles").upsert({
    id:currentUser.id,
    display_name:displayName.value,
    avatar_path:avatarPath
  });

  if(error) return alert(error.message);

  alert("Profile saved");
  if(avatarPath) {
    const {data} = await db.storage.from("user-avatars")
      .createSignedUrl(avatarPath,3600);
    profilePhoto.src = data.signedUrl;
  }
}

async function checkAdmin() {
  const {data,error} = await db.from("admin_users")
    .select("user_id")
    .eq("user_id",currentUser.id)
    .maybeSingle();

  if(!error && data) {
    admin.hidden = false;
    const result = await db.from("profiles")
      .select("id",{count:"exact",head:true});
    users.textContent = result.count ?? 0;
  }
}

// Restore active login after page refresh.
(async()=>{
  const {data} = await db.auth.getSession();
  if(data.session) {
    currentUser = data.session.user;
    await startApp();
  }
})();

create table if not exists public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  display_name text default '',
  avatar_path text,
  created_at timestamptz default now()
);

create table if not exists public.folders (
  id uuid primary key default gen_random_uuid(),
  owner_id uuid references auth.users(id) on delete cascade not null,
  name text not null,
  created_at timestamptz default now(),
  unique(owner_id,name)
);

create table if not exists public.documents (
  id uuid primary key default gen_random_uuid(),
  owner_id uuid references auth.users(id) on delete cascade not null,
  storage_path text unique not null,
  file_name text not null,
  mime_type text,
  size_bytes bigint default 0,
  created_at timestamptz default now()
);

create table if not exists public.admin_users (
  user_id uuid primary key references auth.users(id) on delete cascade
);

alter table public.profiles enable row level security;
alter table public.folders enable row level security;
alter table public.documents enable row level security;
alter table public.admin_users enable row level security;

create policy "profile own"
on public.profiles for all to authenticated
using(id=auth.uid()) with check(id=auth.uid());

create policy "folder own"
on public.folders for all to authenticated
using(owner_id=auth.uid()) with check(owner_id=auth.uid());

create policy "document own"
on public.documents for all to authenticated
using(owner_id=auth.uid()) with check(owner_id=auth.uid());

create policy "admin own read"
on public.admin_users for select to authenticated
using(user_id=auth.uid());

insert into storage.buckets(id,name,public,file_size_limit)
values('user-documents','user-documents',false,52428800)
on conflict(id) do update set public=false,file_size_limit=52428800;

insert into storage.buckets(id,name,public,file_size_limit)
values('user-avatars','user-avatars',false,3145728)
on conflict(id) do update set public=false,file_size_limit=3145728;

create policy "document upload own"
on storage.objects for insert to authenticated
with check(bucket_id='user-documents' and
(storage.foldername(name))[1]=auth.uid()::text);

create policy "document read own"
on storage.objects for select to authenticated
using(bucket_id='user-documents' and
(storage.foldername(name))[1]=auth.uid()::text);

create policy "document delete own"
on storage.objects for delete to authenticated
using(bucket_id='user-documents' and
(storage.foldername(name))[1]=auth.uid()::text);

create policy "avatar upload own"
on storage.objects for insert to authenticated
with check(bucket_id='user-avatars' and
(storage.foldername(name))[1]=auth.uid()::text);

create policy "avatar read own"
on storage.objects for select to authenticated
using(bucket_id='user-avatars' and
(storage.foldername(name))[1]=auth.uid()::text);

create policy "avatar delete own"
on storage.objects for delete to authenticated
using(bucket_id='user-avatars' and
(storage.foldername(name))[1]=auth.uid()::text);
