<script>
	import {tsvParse, tsvFormat,csvFormat,autoType} from 'd3-dsv';
	import BigNumber from "bignumber.js";
	let csv = '';
	let data = [];
	let columns=[];
	let tidy=[];
	let name = 'category';
	let tidycolumns=[];
	let href ='';
	let repeat = null;
	let repeatCheck = false;
	let repeatCol = null;
	
	
	
	function readthedata(csv){
 		data=tsvParse(csv,autoType)
		columns = data.columns
	}

	
	function selectAll(){
		var myNodeList = document.querySelectorAll("input[name=checkboxes]");
		for (let i = 1; i <	myNodeList.length; i++) {
			myNodeList[i].checked = true;
		}
		tidydata(data)
	}

	function tidydata(data){
// 		find columns checked
	 	var checked=[];	
		document.querySelectorAll('input[name=checkboxes]:checked').forEach(function(d){
			checked.push(d.value)
		});
// 		keep unchecked columns
		var notChecked=[];
		document.querySelectorAll('input[name=checkboxes]:not(:checked)').forEach(function(d){
			notChecked.push(d.value)
		});

		
// 		find repeated column
		if(repeatCheck){
			repeatCol = document.querySelector('input[name=repeat]:checked').value		

			const index = notChecked.indexOf(repeat);
			if (index > -1) {
				notChecked.splice(index, 1);
			}
		} else {
			repeatCol = null;
		}	
		
console.log(repeatCheck,repeat, checked,notChecked)
	
		var tidied=[]
		data.forEach(function(e){
			checked.forEach(function(d){
// 				for each checked column, put it in an object
				var obj = {}
				obj[name]=d
				notChecked.forEach(function(d){
					obj[d]=e[d]
				})
				obj["value"]=e[d]
				if(repeat){obj["series"]=d}
				tidied.push(obj)
				
// 				now do it for the repeated column
				if(repeat){
					var repeatedObj={}
					repeatedObj[name]=d
					notChecked.forEach(function(d){
						repeatedObj[d]=e[d]
					})
					repeatedObj['value']=e[repeatCol]
					repeatedObj['series']=repeatCol
					tidied.push(repeatedObj)
				}
				
			})	
		})
		href=encodeURI('data:text/csv;charset=utf-8,'+csvFormat(tidied))
		tidy = tsvParse(tsvFormat(tidied))
		tidycolumns = tidy.columns
	}
	
	function timesTen() {
  tidy.forEach(d => {
    if (!BigNumber(d.value).isNaN()) {
      d.value = BigNumber(d.value).times(10);
    }
  });
  tidy = tidy;
}

function divideTen() {
  tidy.forEach(d => {
    if (!BigNumber(d.value).isNaN()) {
      d.value = BigNumber(d.value).div(10);
    }
  });
  tidy = tidy;
}

$:if(repeat&&repeatCheck)tidydata(data);

$:if(repeatCheck==false)repeat=null;

</script>

<h1>Let's tidy your data!</h1>
<h2>
	Step 1. Paste your data
</h2>
<label for="csvdata">Paste data in from Excel</label><br/>
<textarea on:input="{e => readthedata(e.target.value)}" rows=10 columns=10 id="csvdata"></textarea>

<p>
	Here's your table
</p>

<div class="tablewrapper">
	<table>
		<thead>
			<tr>
			{#each columns as d}
			<th>{d}</th>
			{/each}
			</tr>
		</thead>
		<tbody>
			{#each data as d}
			<tr>
				{#each Object.keys(d) as e}
				<td>{d[e]}</td>
				{/each}
			</tr>
			{/each}
		</tbody>
	</table>
</div>

<h2>
	Step 2: Choose your columns to merge
</h2>


	

<div>
<p>
	Choose which columns to combine
</p>

<button on:click={selectAll}>
	Select all except first column
</button>

	{#each columns as d}
	<div class="item">
		<input name="checkboxes" type="checkbox" on:input={e=>tidydata(data)} id={d} value={d}><label for={d}>{d}</label>
	</div>
	{/each}
	
</div>
<br/>
<h4>Optional</h4>
<div>
	<label for="repeatcheck">I need to repeat a column.</label> <input id="repeatcheck" type="checkbox" bind:checked={repeatCheck}>

	{#if repeatCheck}
		<p>
			Which column to repeat?
	</p>
		{#each columns as d}
		<div class="item">
			<input name="repeat" type="radio" on:input={e=>tidydata(data)} bind:group={repeat} id={'radio'+d} value={d}><label for={'radio'+d}>{d}</label>
		</div>
		{/each}
	{/if}
</div>


<br/>
<div>
<label for="columnname">
	Choose a new column name 
</label>
<input id="columnname" on:input={e=>tidydata(data)} bind:value={name} type="text">
	</div>
<h2>
	Step 3. Check your tidy data
</h2>
<p>
	Here's your tidy data
</p>
<button on:click={timesTen}>Multiply values by 10</button>
<button on:click={divideTen}>Divide values by 10</button>
<!-- These buttons aren't working yet, struggling to get string to convert to numbers -->
<div class="tablewrapper">
	<table>
		<thead>
			<tr>
			{#each tidycolumns as d}
			<th>{d}</th>
			{/each}
			</tr>
		</thead>
		<tbody>
			{#each tidy as d}
			<tr>
				{#each Object.keys(d) as e}
				<td>{d[e]}</td>
				{/each}
			</tr>
			{/each}
		</tbody>
	</table>
</div>

<h2>
	Step 4. Download your data
</h2>
<p>
	Happy with your new data? Let's download it
</p>
<button>
	<a href={href} download='data.csv'>Download as .csv</a>
</button>

<style>
	.tablewrapper{
		height:150px;
		overflow:scroll
	}
	
	thead tr {
    background-color: #206095;
    color: #ffffff;
    text-align: left;
}
	th, td {
    padding: 5px;
	}
	
	tbody tr {
			border-bottom: 1px solid #dddddd;
	}

	tbody tr:nth-of-type(even) {
			background-color: #f3f3f3;
	}

	tbody tr:last-of-type {
			border-bottom: 2px solid #206095;
	}
	.item{
		display:block;
	}
	
	.item *{
		display:inline;
		padding: 5px;
	}
	
	
</style>