# Mentions
%%
has_mention_list:: 1
%%
```dataviewjs
// #👨‍💻/dataviewjs/mentions/1

// ================================== USER CONDITIONS ==============================
let use_tables = true;
let columns = 2;

let user_field__hierarchy = "hierarchy_in_mentions__";
let usr__use_old_code_for_rendering = true;
let EXCLUDE_DATAVIEW_FILES = true;
let SHOW_ONLY_SPECIFIC_FIELD = true;
let usr_field = "summary";
let SHORT_BASED_ON = 1; //1: hierarchy in mentions, 2: recency
let AESTH__SHOW_NOTE_AS_HEADER = true;


// Formatting choices
let put_summaries_into_admonition_blocks = true;
let admonition_block_for_summary = "note";
let all_blocks_in_quote = true;
 
var notesToGiveOnlyLinks = [
    "p770", "First working document", "Marios-Bjarne--23-08-30", 
    "Master Student Simon Physics-Based MTL", "Hedvig", 
    "✍⌛writing--PHEN--loss-landscapes", 
    "✍⌛writing--WDOC1--quantities-factors we can use to characterize MTL Dynamics"
];

var excludeFolder = ["👨‍💻Automations", "🔎 Traceability of literature reading"];
var excludeFilesWithNamesStartingWith = ["datav"]; // DO NOT ADD "datav"!

// ==============================================================================


// =================== CREATE CONDITION BUTTON =========================

const {createButton} = app.plugins.plugins["buttons"]
const tp = app.plugins.plugins["templater-obsidian"].templater.current_functions_object

let current = dv.current()
let currentFile = app.vault.getAbstractFileByPath(current.file.path)


const changeCnd = async(cnd) => {
  let propName = "condition__prioritize_tasks"
  current[propName] = cnd
  dv.el("article", cnd)
  app.fileManager.processFrontMatter(currentFile, (frontmatter) => { 
		frontmatter[propName] = cnd
	})
  } 


const conditionButton = async(name, cnd) => {
  createButton({
	app, 
	el: this.container, 
	args: {
		name: name}, 
	clickOverride: {
		click: await changeCnd, 
		params: [cnd]
	}
  })
}

let cnd_field = "condition__prioritize_tasks";
let usr__filter__prioritize_tasks = current[cnd_field];

let button_msg = '';
if (usr__filter__prioritize_tasks){
	button_msg = "Deactivate Prioritization of notes with tasks";
} else {
	button_msg = "Activate Prioritization of notes with tasks";
}
await conditionButton(button_msg, !current[cnd_field])
// ========================================================


// Create message about user preferences
var msg__print_user = 'Showing notes';
if (usr__filter__prioritize_tasks){
	msg__print_user += ' prioritized on task-notes.';
}


// ========================== DEF FUNCTIONS ====================================

async function generateTable(columns, text_render) {
    let tableRows = [];
    let row = [];
    var added_text = '';
    for (let i = 0; i < text_render.length; i++) {
        let cellRenderInstructions = text_render[i];
        // Initialize a string to accumulate the HTML content for the cell
        let cellContent = document.createElement("div");

        // Generate the HTML content for the cell
        for (let t_i of cellRenderInstructions) {
            // Check if the content is a link element
            if (t_i[1].length>0){
	            let tmp1 = t_i[1] + '\n';
		        if (t_i[0][0]=='h'){
			        let header_level = t_i[0][1];
				    if (header_level==1){
					    added_text = '# ' + tmp1; 
					} else if (header_level==2){
						added_text = '## ' + tmp1; 
					} else if (header_level==3){
						added_text = '### ' + tmp1; 
					} else if (header_level==4){
						added_text = '#### ' + tmp1; 
					} else if (header_level==5){
						added_text = '##### ' + tmp1; 
					} else if (header_level==6){
						added_text = '###### ' + tmp1; 
				    }
				} else {
					added_text = tmp1; 	 
		        }
		        cellContent.innerHTML += added_text;
	        }
        }

        // Add the accumulated cell content to the row
        row.push(cellContent.innerHTML);

        // If we've reached the number of columns, push the row and reset it
        if ((i + 1) % columns == 0) {
            tableRows.push(row);
            row = [];
        }
    }

    if (row.length > 0) {
        tableRows.push(row);
    }

    return tableRows;
}


function renderNoteWithHierarchy_get_render_list(p, show_note_link, user_field__hierarchy, AESTH__SHOW_NOTE_AS_HEADER) {
    var type_of_render = '';
    let header_render = p + ", H = " + show_note_link;
    if (AESTH__SHOW_NOTE_AS_HEADER) {
        type_of_render = 'h4';
    } else {
        type_of_render = 'article';
    }
	return [type_of_render, header_render];
}

function isArrayofStrings(variable) { 
	return Array.isArray(variable) && variable.every(element => typeof element === 'string'); 
}

function renderNoteContent_append(showOnlyLink, SHOW_ONLY_SPECIFIC_FIELD, dvpages, usr_field, noteContentList, idx_p) {
	
	var list_render = [];

	if (put_summaries_into_admonition_blocks){
		var start_ad_block = '```ad-'+ admonition_block_for_summary + '\n';
		var end_ad_block = '```';
	} else {
		var start_ad_block = '';
		var end_ad_block = '';
	}

    if (!showOnlyLink) {
        if (SHOW_ONLY_SPECIFIC_FIELD) {
            let u_fld = dvpages[usr_field];
            if (u_fld.length > 0) {
	            if (isArrayofStrings(u_fld)){
		            var u_fld_text = start_ad_block;
		            for (let u_fld_i of u_fld) {
			            u_fld_text += '###### ' + u_fld_i + '\n'; 
					}
					u_fld_text += end_ad_block;
		            list_render.push(['article', u_fld_text]);
				} else {
		            list_render.push(['h6', u_fld]);
				}
				list_render.push(['h6', dvpages.comment_for_project]);
            } else {
	            if (all_blocks_in_quote){
		            var start_ad_block = '```ad-quote\n';
		            var end_ad_block = '\n```';
				} else {
		            var start_ad_block = '';
		            var end_ad_block = '';
	            }
	            list_render.push(['h6', dvpages.comment_for_project]);
				list_render.push(['article', start_ad_block + noteContentList[idx_p] + end_ad_block]);
            }
        } else {
	        list_render.push(['article', noteContentList[idx_p]]);
        }
    }
	return list_render;
}


// ==============================================================================


if (SHORT_BASED_ON==1){
	var inlinks =  dv.current().file.inlinks.sort(l => dv.pages('"' + l.path +'"')[user_field__hierarchy], "desc").sort(l => dv.pages('"' + l.path +'"').file.cdate, "asc");
	var short_based_on_msg = '\n' + '```\n' + user_field__hierarchy + '\n' + '```';
} else if (SHORT_BASED_ON==2){
	var inlinks = Array.from(dv.current().file.inlinks).reverse();
	var short_based_on_msg = "recency";
}

dv.el("h6", "Number of Mentions: " + inlinks.length);
dv.el("h6", "Sorted based on: " + short_based_on_msg);

var flippedInlinks = inlinks;
var text_print;

if (usr__filter__prioritize_tasks){
	// Separate links with tasks and without tasks
	let linksWithTasks = [];
	let linksWithoutTasks = [];
	
	for (let link of flippedInlinks) {
	    let page = dv.page(link.path);
	    if (page) {
	        // Load the content of the page
	        let content = await dv.io.load(link.path);
	        // Check if the content contains task syntax
	        let hasTasks = content.includes('- [ ]') || content.includes('➕');// || content.includes('- [x]');
	        if (hasTasks) {
	            linksWithTasks.push(link);
	        } else {
	            linksWithoutTasks.push(link);
	        }
	    } else {
	        linksWithoutTasks.push(link);
	    }
	}
	// Combine links with tasks first, then links without tasks
	flippedInlinks = [...linksWithTasks, ...linksWithoutTasks];
}
let L = flippedInlinks.length;

// Create a list of "true" and "false"  
var displayContentList = flippedInlinks.map(link => {
    let noteName = link.path.split('/').pop().replace(/\.md$/, '');
    return !notesToGiveOnlyLinks.includes(noteName);
});

// Create a list of "true" and "false" based on outlink condition
var wasAddedToPhenomenaReportList = await Promise.all(flippedInlinks.map(async (link) => {
    let page = dv.page(link.path);
    if (page) {
        let outlinks_ik = page.file.outlinks;
        for (let outlink of outlinks_ik) {
            if (outlink.path.includes("✔added to report PHEN--done")) {
                return true;
            }
        }
    }
    return false;
}));

// Create a list of "true" and "false" based on excludeFolder condition
var excludeFolderList = flippedInlinks.map(link => {
    return excludeFolder.some(folder => link.path.includes(folder));
});


// Create a list to hold note content
var noteContentList = await Promise.all(flippedInlinks.map(async (link) => {
    let content = await dv.io.load(link.path);
    return content;
}));

// Create a boolean list based on excludeFilesWithNamesStartingWith condition
var excludeFilesList_folder_starts_with = flippedInlinks.map(link => {
    return !excludeFilesWithNamesStartingWith.some(prefix => link.path.startsWith(prefix));
});




var text_render = [];

let L_excl_folder = excludeFolder.length;
let L_notesToGiveOnlyLinks = notesToGiveOnlyLinks.length;

if (EXCLUDE_DATAVIEW_FILES) {
	excludeFilesWithNamesStartingWith.push("datav");
} else {
	// Check if "datav" exists in the array and remove it if found
	const index = excludeFilesWithNamesStartingWith.indexOf("datav");
	if (index !== -1) {
		excludeFilesWithNamesStartingWith.splice(index, 1);
	}
}
var was_added_to_phenomena_report = false;
var idx_p = 0; //iteratable index 
for (let p of flippedInlinks) { // Loop through pages 
	
	var text_render_p = [];
	let ppath = p.path;
	let dvpages = dv.pages('"' + ppath +'"');
	let excludeNote = false;
	let showOnlyLink = false;

	excludeNote = excludeFolderList[idx_p];

	// to avoid the problem of getting stuck
	if (dvpages.has_mention_list.length > 0){showOnlyLink = true}
	
	if (!excludeNote) {
		// Check if the note's name starts with any excluded string

		excludeNote = !excludeFilesList_folder_starts_with[idx_p];
		if (!excludeNote) {
			text_render_p.push(renderNoteWithHierarchy_get_render_list(p, dvpages[user_field__hierarchy], user_field__hierarchy, AESTH__SHOW_NOTE_AS_HEADER));


			showOnlyLink = !displayContentList[idx_p]

			if (wasAddedToPhenomenaReportList[idx_p]){
				text_render_p.push(["h5", "✔Phenomena Report"]);
			}
			text_render_p.push(...renderNoteContent_append(showOnlyLink, SHOW_ONLY_SPECIFIC_FIELD, dvpages, usr_field, noteContentList, idx_p));
			text_render_p.push(["article", "\n---"]);
		}
	}

	text_render.push(text_render_p)
	idx_p += 1
}

// RENDER NOTES
if (!use_tables){
	for (let t of text_render) {
		for (let t_i of t) {
			dv.el(t_i[0], t_i[1]);
		}
	}
} else {

	// Call the function and render the table		
	generateTable(columns, text_render).then(tableRows => {
		let tableHeader = Array(columns).fill("Content");
	
		// Render the table
		dv.table(tableHeader, tableRows);
	});
}
//

```