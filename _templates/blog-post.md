<%*
// 제목 입력 받기
const title = await tp.system.prompt("제목을 입력하세요");
if (!title) return;

// 카테고리 선택 (텍스트와 함께)
const CATEGORIES = ["devlogs", "projects", "journals"];
const category = await tp.system.suggester(
   CATEGORIES,
   CATEGORIES,
   "카테고리를 선택하세요"
);
if (!category) return;

// 폴더 선택 옵션
const folders = ["_drafts", "_posts/devlogs", "_posts/projects", "_posts/journals"];
const targetFolder = await tp.system.suggester(
   folders,
   folders,
   "저장할 폴더를 선택하세요"
);
if (!targetFolder) return;

// 주요 내용 목록 입력 받기 (여러 줄 입력 가능)

let contentItems = [];
let keepAsking = true;
let itemNumber = 1;
while (keepAsking) { 
	const item = await tp.system.prompt(`주요 내용 #${itemNumber} (완료시 빈 값 입력)`);
	if (item) {
	 contentItems.push(`- ${item}`); itemNumber++; 
	 } 
	 else {
	  keepAsking = false; 
	}
}


// 파일명 포맷팅
const date = tp.date.now("YYYY-MM-DD");
const formattedTitle = title
   .toLowerCase()
   .replace(/\s+/g, '-')
   .replace(/[^a-z0-9-가-힣]/g, '')
   .replace(/-+/g, '-');
const newFileName = `${date}-${formattedTitle}`;

// 태그 입력 받기 (쉼표로 구분)
const tags = await tp.system.prompt("Tags (쉼표로 구분)", "tag1, tag2");
const formattedTags = tags ? tags.split(',').map(tag => tag.trim()) : [];

// 폴더가 없으면 생성
if (!tp.file.exists(targetFolder)) {
   await app.vault.createFolder(targetFolder);
}

// 파일 이동 및 이름 변경
await tp.file.move(`${targetFolder}/${newFileName}`);
-%>
---
title: "<% title %>"
date: <% tp.date.now("YYYY-MM-DD HH:mm:ss") %> +0900
categories: [<% category %>]
tags: [<% formattedTags.join(', ') %>]
imageNameKey:  "<% newFileName %>"
---

<% contentItems.length > 0 ? `
>${contentItems.join('\n')} 
{: .prompt-info }
` : '' %>

