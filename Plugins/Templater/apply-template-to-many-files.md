---
categories:
  - plugin-templater
description: 
title: Применить шаблон ко всем файлам лажащим в папке
updatedAt: 2025-05-13T23:11:47
createdAt: 2025-05-13T19:56:51
---
**Задача**, нужно применить темплейт плагина Templater [GitHub - SilentVoid13/Templater: A template plugin for obsidian](https://github.com/SilentVoid13/Templater) для всех файлов в директории и поддиректориях.

``` js
<%* 
// === Настройки ===

// Папка, в которой выбираются файлы для применения шаблона
const targetPath = '00.INBOX';

// Путь к файлу-шаблону Templater (используется только для формирования имени команды)
const templatePath = "07.Meta/01.Templates/01.Actions/created_note.md";

// Количество обрабатываемых файлов (для начала — 1, чтобы проверить корректность)
const countFiles = 1;

// Время ожидания после запуска шаблона (в миллисекундах)
const waitMs = 1000;

// === Утилита ожидания ===
function sleep(ms: number) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// === Получение нужных файлов ===
const files = app.vault.getMarkdownFiles()
  .filter(file => file.path.startsWith(`${targetPath}/`))
  .slice(0, countFiles);

// === Инициализация интерфейса ===
const { workspace } = app;
const leaf = workspace.getLeaf(true);

// Имя команды шаблона Templater
const commandName = "templater-obsidian:" + templatePath;

for (const file of files) {
  console.log("Обрабатывается файл:", file.path);

  // Открываем файл во вкладке
  await leaf.openFile(file);
  await workspace.setActiveLeaf(leaf, true);

  // Ставим курсор в конец документа
  const editor = leaf.view?.editor;
  if (editor) {
    const lastLine = editor.lastLine();
    const lastCh = editor.getLine(lastLine).length;
    editor.setCursor({ line: lastLine, ch: lastCh });
  }

  // Уведомляем пользователя
  new Notice(`Файл "${file.path}" начал обработку`);

  // Запускаем команду шаблона
  const command = app.commands.commands[commandName];
  if (command?.callback) {
    command.callback();
  } else {
    new Notice(`Команда шаблона не найдена ${commandName}. Неверный ${templatePath}.`);
  }

  // Ждём перед переходом к следующему файлу
  await sleep(waitMs);
}
%>
```