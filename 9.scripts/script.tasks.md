<%*
const quickAddApi = app.plugins.plugins.quickadd.api;

// --- CONSTANTS ---

const DATE_FORMAT = "YYYY-MM-DD";

const UNFINISHED_TAGGED_TASK_REGEX = /^\s*-\s\[ \]\s.*#task\b/;
const DUE_DATE_REGEX = /📅\s*\d{4}-\d{2}-\d{2}/;

// --- PROMPTS ---

async function promptDueDate() {
    const today = moment().format(DATE_FORMAT);
    const input = await quickAddApi.inputPrompt("Due date (YYYY-MM-DD)", today, today);
    if (!input) return null;

    if (!moment(input, DATE_FORMAT, true).isValid()) {
        new Notice(`Invalid date: ${input}`);
        return null;
    }

    return input;
}

// --- TASK HELPERS ---

function isUnfinishedTaggedTask(line) {
    return UNFINISHED_TAGGED_TASK_REGEX.test(line);
}

function withDueDate(line, date) {
    const dueDateText = `📅 ${date}`;

    if (DUE_DATE_REGEX.test(line)) {
        return line.replace(DUE_DATE_REGEX, dueDateText);
    }

    return `${line.trimEnd()} ${dueDateText}`;
}

async function updateDueDates(file, date) {
    const content = await app.vault.read(file);
    const lines = content.split("\n");

    let updatedCount = 0;
    const updatedLines = lines.map((line) => {
        if (!isUnfinishedTaggedTask(line)) return line;
        updatedCount++;
        return withDueDate(line, date);
    });

    await app.vault.modify(file, updatedLines.join("\n"));
    return updatedCount;
}

// --- MAIN EXECUTION ---

const file = tp.config.target_file;

const date = await promptDueDate();
if (!date) return;

const updatedCount = await updateDueDates(file, date);

new Notice(`Due date set to ${date} for ${updatedCount} unfinished task(s).`);
%>
