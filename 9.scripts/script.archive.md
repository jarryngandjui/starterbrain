<%*
const quickAddApi = app.plugins.plugins.quickadd.api;
const now = moment().format("YYYY-MM-DDTHH:mm");
const ARCHIVE_ROOT = "10.archive";

const CURRENT_FILE_LABEL = "current file";

const FOLDER_OPTIONS = [
    { label: "blog", path: "7.zettelkasten/blog" },
    { label: "content", path: "7.zettelkasten/content" },
    { label: CURRENT_FILE_LABEL, path: null },
    { label: "daily", path: "1.daily" },
    { label: "external", path: "2.external" },
    { label: "sprint", path: "7.zettelkasten/sprint" },
    { label: "workout", path: "7.zettelkasten/workout" },
];

// --- MODULAR FUNCTIONS ---

async function promptFolders() {
    const labels = FOLDER_OPTIONS.map((o) => o.label);
    const chosen = await quickAddApi.checkboxPrompt(labels, []);
    return FOLDER_OPTIONS.filter((o) => chosen.includes(o.label));
}

async function promptCutoverDate() {
    const thisYear = Number(moment().format("YYYY"));
    const years = Array.from({ length: 6 }, (_, i) => String(thisYear - i));
    const year = await quickAddApi.suggester(years, years);
    if (!year) return null;

    const months = moment.months().map((name, i) => ({
        label: name,
        value: String(i + 1).padStart(2, "0"),
    }));
    const month = await quickAddApi.suggester(
        months.map((m) => m.label),
        months.map((m) => m.value)
    );
    if (!month) return null;

    const daysInMonth = moment(`${year}-${month}`, "YYYY-MM").daysInMonth();
    const days = Array.from({ length: daysInMonth }, (_, i) => String(i + 1).padStart(2, "0"));
    const day = await quickAddApi.suggester(days, days);
    if (!day) return null;

    return moment(`${year}-${month}-${day}`, "YYYY-MM-DD");
}

function findMatches(folders, cutoverDate) {
    const allFiles = app.vault.getFiles();
    const matches = [];

    for (const folder of folders) {
        const prefix = folder.path + "/";
        const files = allFiles.filter((f) => f.path.startsWith(prefix));
        for (const file of files) {
            const created = moment(file.stat.ctime);
            if (created.isBefore(cutoverDate, "day")) {
                matches.push({ file, label: folder.label });
            }
        }
    }

    return matches;
}

function formatMatchesPreview(matches, maxPerGroup = 10) {
    // U+2028 forces a real line break in QuickAdd's plain-text modal,
    // where a normal "\n" would just collapse into a space.
    const NEWLINE = " ";
    const byLabel = new Map();
    for (const match of matches) {
        if (!byLabel.has(match.label)) byLabel.set(match.label, []);
        byLabel.get(match.label).push(match.file);
    }

    const groups = [];
    for (const [label, files] of byLabel) {
        const shown = files.slice(0, maxPerGroup).map((f) => `  • ${f.basename}`);
        const rest = files.length > maxPerGroup ? [`  ...and ${files.length - maxPerGroup} more`] : [];
        groups.push([`${label} (${files.length})`, ...shown, ...rest].join(NEWLINE));
    }

    return groups.join(NEWLINE + NEWLINE);
}

async function confirmMatches(matches, cutoverDate) {
    const dateLabel = cutoverDate ? ` created before ${cutoverDate.format("YYYY-MM-DD")}` : "";
    return await quickAddApi.yesNoPrompt(
        `Archive ${matches.length} file(s)${dateLabel}?`,
        formatMatchesPreview(matches)
    );
}

function getTargetPath(file) {
    const year = moment(file.stat.ctime).format("YYYY");
    const parentPath = file.parent && file.parent.path !== "/" ? file.parent.path : "";
    const targetFolder = parentPath
        ? `${ARCHIVE_ROOT}/${parentPath}/${year}`
        : `${ARCHIVE_ROOT}/${year}`;
    return { targetFolder, targetPath: `${targetFolder}/${file.name}` };
}

async function ensureFolderExists(folderPath) {
    // Creates every missing segment of a (possibly nested) folder path one
    // level at a time, since files being archived may live several folders
    // deep and the mirrored archive path won't exist yet at any level.
    const segments = folderPath.split("/");
    let current = "";
    for (const segment of segments) {
        current = current ? `${current}/${segment}` : segment;
        if (!app.vault.getAbstractFileByPath(current)) {
            await app.vault.createFolder(current);
        }
    }
}

async function archiveFile(file) {
    if (file.path.startsWith(ARCHIVE_ROOT + "/")) return false;

    const { targetFolder, targetPath } = getTargetPath(file);
    if (app.vault.getAbstractFileByPath(targetPath)) {
        console.warn(`Skipping "${file.path}": target already exists at "${targetPath}".`);
        return false;
    }

    if (file.extension === "md") {
        await app.fileManager.processFrontMatter(file, (frontmatter) => {
            frontmatter["archive date"] = now;
        });
    }

    await ensureFolderExists(targetFolder);
    await app.fileManager.renameFile(file, targetPath);
    return true;
}

// --- MAIN EXECUTION ---

const selections = await promptFolders();
if (selections.length === 0) {
    new Notice("Aborting: no folders selected.");
    return;
}

const archiveCurrentFile = selections.some((o) => o.label === CURRENT_FILE_LABEL);
const folders = selections.filter((o) => o.path !== null);

let cutoverDate = null;
if (folders.length > 0) {
    cutoverDate = await promptCutoverDate();
    if (!cutoverDate) return;
}

const matches = folders.length > 0 ? findMatches(folders, cutoverDate) : [];
if (archiveCurrentFile) {
    matches.push({ file: tp.config.target_file, label: CURRENT_FILE_LABEL });
}

if (matches.length === 0) {
    new Notice("Nothing to archive: no files created before that date.");
    return;
}

const confirmed = await confirmMatches(matches, cutoverDate);
if (!confirmed) {
    new Notice("Aborting: archive cancelled.");
    return;
}

let archivedCount = 0;
for (const match of matches) {
    if (await archiveFile(match.file)) archivedCount++;
}

new Notice(`Archived ${archivedCount} file(s).`);
%>
