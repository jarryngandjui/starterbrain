<%*
const quickAddApi = app.plugins.plugins.quickadd.api;
const file = tp.config.target_file;

// --- TEMPLATE CONFIGURATIONS ---

const CONFIGS = {
    big7: {
        "kind": ["sprint"],
        "status": ["🔲 backlog"],
        "category": ["planning"],
        "story": "",
        "complete": false,
        "tags": ["template", "big7"],
        "due date": "",
        "completed date": "",
        "start date": "",
        "cancelled date": "",
        "cover": "",
        "rating": "",
        "notetoolbar": "sprint"
    },
    content: {
        "kind": ["story"],
        "status": ["🔲 backlog"],
        "project": "[[Brand]]",
        "story": "",
        "complete": false,
        "tags": ["template", "content/tiktok", "content/youtube", "content/instagram", "content"],
        "due date": "",
        "completed date": "",
        "start date": "",
        "cancelled date": "",
        "instagram": "",
        "tiktok": "",
        "youtube": "",
        "category": ["brand"],
        "notetoolbar": "default"
    },
    person: {
        "id": "Person Template",
        "aliases": [],
        "tags": ["template", "person/template"],
        "category": ["social"],
        "complete": false,
        "kind": ["story"],
        "notetoolbar": "default",
        "status": ["🔲 backlog"]
    },
    external: {
        "id": "",
        "cover": "",
        "kind": ["story"],
        "status": ["🔲 backlog"],
        "project": "",
        "story": "",
        "complete": false,
        "category": ["finance", "health", "brand", "planning", "social", "engineer"],
        "tags": ["template", "external", "pacer"],
        "url": "",
        "links": "",
        "due date": "",
        "completed date": "",
        "start date": "",
        "cancelled date": "",
        "notetoolbar": "default",
        "productivity": ""
    },
    workout: {
        "notetoolbar": "sprint",
        "id": "",
        "aliases": [],
        "kind": ["sprint"],
        "category": ["health"],
        "status": ["🔲 backlog"],
        "cover": "",
        "project": "",
        "story": "",
        "url": "",
        "links": "",
        "tags": ["workout", "template"],
        "complete": false,
        "completed date": "",
        "start date": "",
        "cancelled date": "",
        "due date": ""
    },
    default: {
        "id": "",
        "cover": "",
        "kind": ["project", "story", "sprint"],
        "status": ["🔲 backlog"],
        "project": "",
        "story": "",
        "complete": false,
        "category": ["finance", "health", "brand", "planning", "social", "engineer"],
        "tags": ["template"],
        "url": "",
        "links": "",
        "due date": "",
        "completed date": "",
        "start date": "",
        "cancelled date": "",
        "notetoolbar": "default",
        "productivity": ""
    }
};

// --- MAIN EXECUTION ---

// Step 1: Choose template configuration
const configOptions = ["big7", "content", "person", "external", "workout", "default"];
const configChoice = await quickAddApi.suggester(configOptions, configOptions);

if (!configChoice) return;

const selectedConfig = CONFIGS[configChoice];

// Step 2: Choose merge strategy
const mergeOptions = ["Add missing properties", "Overwrite all properties", "Cancel"];
const mergeChoice = await quickAddApi.suggester(mergeOptions, mergeOptions);

if (!mergeChoice || mergeChoice === "Cancel") return;

// Step 3: Apply the configuration
await app.fileManager.processFrontMatter(file, (frontmatter) => {
    if (mergeChoice === "Overwrite all properties") {
        // Replace all properties with template config
        Object.keys(frontmatter).forEach(key => delete frontmatter[key]);
        Object.assign(frontmatter, selectedConfig);
    } else if (mergeChoice === "Add missing properties") {
        // Only add properties that don't exist
        for (const [key, value] of Object.entries(selectedConfig)) {
            if (!(key in frontmatter)) {
                frontmatter[key] = value;
            }
        }
    }
});

new Notice(`${configChoice} properties ${mergeChoice === "Overwrite all properties" ? "applied" : "merged"}`);
%>
