<%*
const dateFormat = "YYYY-MM-DD";
const todayDate = moment().format(dateFormat);
const dailyFolder = "1.daily";
const targetPath = `${dailyFolder}/${todayDate} Daily Note.md`;
const thisFile = tp.config.target_file;

const existingFile = app.vault.getAbstractFileByPath(targetPath);

if (existingFile && existingFile.path !== thisFile.path) {
    await app.workspace.getLeaf(false).openFile(existingFile);
    new Notice(`Opened existing daily note: ${todayDate}`);
    setTimeout(() => app.vault.delete(thisFile), 200);
    return;
}

const expectedBasename = `${todayDate} Daily Note`;
if (thisFile.basename !== expectedBasename) {
    await tp.file.rename(expectedBasename);
}
%>
