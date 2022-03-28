###BLAZOR 6 templated table with column hiding and column ordering.

An important part of this solution is the use of RenderFragments defined in code.
I found documentation on how to do that here [here](https://docs.microsoft.com/en-us/aspnet/core/blazor/performance?view=aspnetcore-6.0#define-reusable-renderfragments-in-code). An Important Note here is that *Assignment to a RenderFragment delegate is only supported in Razor component files (.razor), and event callbacks aren't supported.* Any suggestions on how to overcome this issue would be much appreciated.


**SharedTestData.cs** dataset for testing purposes
```csharp
namespace BlazorTestSandbox.Client.Shared.DataClass
{
    public class SharedTestData
    {
        public string id { get; set; }
        public string name { get; set; }
        public string description { get; set; }
        public DateTime executed_on { get; set; }
        public string executed_by { get; set; }
        public bool success { get; set; }
        public decimal error_CF { get; set; }
    }
}
```
****
**MVCColumnCtrl.cs** to be instantiated per column in the sample data dataset(`SharedTestData`)
```csharp
using Microsoft.AspNetCore.Components;

namespace BlazorTestSandbox.Client.Shared.MVCTable
{
    public class MVCColumnCtrl<TItem>
    {
        public string label { get; set; }
        public string id { get; set; }
        public bool visible { get; set; } = true;
        public RenderFragment<TItem> RenderValue { get; set; }
        public MVCColumnCtrl(
            string id, RenderFragment<TItem> RenderValue, string label = ""
        )
        {
            this.id = id;
            if (label == "")
            {
                this.label = id;
            } else { 
                this.label = label;
            }
            this.RenderValue = RenderValue;
        }
    }
}
```
****
**MVCTable.razor** iterates and renders data and columns, receives `MVCColumnCtrl` array and `SharedTestData` array from `TableTest.razor` page
```csharp
@using BlazorTestSandbox.Client.Shared.AppStates
@using BlazorTestSandbox.Client.Shared
@using BlazorTestSandbox.Client.Shared.DataClass

@typeparam TItem

<table class="table  table-striped table-sm">
    <thead>
        <tr>
        @foreach(MVCColumnCtrl<TItem> Col in visible_col_names)
        {
            <th scope="col" >@Col.label</th>
        }
        </tr>
    </thead>
    <tbody>
            @foreach (TItem item in data)
            {
                <tr>
                @foreach(MVCColumnCtrl<TItem> Col in visible_col_names)
                {
                    @Col.RenderValue(item)
                }
                </tr>
            }
    </tbody>
</table>

@code {
    [Parameter]
    public IEnumerable<TItem> data { get; set; }

    [Parameter]
    public MVCColumnCtrl<TItem>[] visible_col_names { get; set; }
}
```
****
**MVCTableColumnManager.razor** renders dropdown menu and adds column hiding and ordering functionality, receives `MVCColumnCtrl` array and `RefreshColumns` Action form `TableTest.razor` page
```csharp
@typeparam TItem

<div 
    class=@(show? "dropdown show" : "dropdown" )
>
  <button class="btn btn-sm btn-outline-primary dropdown-toggle" type="button" id="dropdownMenu2" data-toggle="dropdown" aria-haspopup="true" 
  aria-expanded=@show
  @onclick=@(()=> @show = !@show )
  >⚙</button>
    <div class=@(show? "dropdown-menu show" : "dropdown-menu" )>
      <div class="px-4 py-3">
         <table class="table  table-striped table-sm">
           <thead>
                <tr>
                    <th scope="col" >#</th>
                    <th scope="col" >Column</th>
                    <th scope="col" >Show</th>
                    <th scope="col" >Order</th>
                </tr>
          </thead>
            <tbody>
               @for (var i = 0; i < mvc_columns.Length ; i++)
               {
                    MVCColumnCtrl<TItem> Col = mvc_columns[i];
                    int row_i = i + 0;
                        <tr>
                    <td>@(row_i+1)</td>
                    <td scope="row" ><b>@Col.label</b> </td>
                    <td class="text-center"><input type="checkbox" @onchange=@(()=>ToggleVisibleCol(@Col) ) checked=@Col.visible  /></td>
                    <td>
                        <button @onclick=@(()=>MoveColUp(@Col, @row_i) ) class="btn-outline-primary pb-0 pt-0" style="border-radius: 3px!important">↑</button>
                        <button @onclick=@(()=>MoveColDown(@Col, @row_i) ) class="btn-outline-primary pb-0 pt-0" style="border-radius: 3px!important">↓</button>
                    </td>
                </tr>
               }
            </tbody>
        </table>

      </div>
    </div>
</div>
@code {
    private bool show  { get; set; } = false;
    [Parameter] public MVCColumnCtrl<TItem>[] mvc_columns { get; set; }
    [Parameter] public Action RefreshColumns { get; set; }
    public void ToggleVisibleCol(MVCColumnCtrl<TItem> Col)
    {
        Col.visible = !Col.visible; 
        RefreshColumns();
    }
    public void MoveColUp(MVCColumnCtrl<TItem> colUp, int i)
    {
        int iUp = 0;
        MVCColumnCtrl<TItem> colDown = null;

        if(i != 0)
        {                    
            iUp = i - 1;
            colDown = mvc_columns[i - 1];
        }
      
        if(colDown != null && colUp != null)
        {            
            mvc_columns[iUp] = colUp;
            mvc_columns[iUp + 1] = colDown;
            RefreshColumns();
        }
    }
    public void MoveColDown(MVCColumnCtrl<TItem> colDown, int i)
    {
        int iDown = 0;
        MVCColumnCtrl<TItem> colUp = null;

        if(i !=  mvc_columns.Length-1)
        {                    
            iDown = i + 1;
            colUp = mvc_columns[i + 1];
        }

        if(colUp != null && colDown != null)
        {            
            mvc_columns[iDown] = colDown;
            mvc_columns[iDown - 1] = colUp;
            RefreshColumns();
        }
    }

}

```
****
**TableTest.razor** here all the models and data are instantiated and passed to components I `MVCTableColumnManager` and `MVCTable`. This should probably be cleaned up a bit.
```csharp
@page "/TableTest"
@using BlazorTestSandbox.Client.Shared.AppStates
@using BlazorTestSandbox.Client.Shared.DataClass
@using BlazorTestSandbox.Client.Shared.MVCTable
<h3>TableTest</h3>

<MVCTableColumnManager 
    TItem=@SharedTestData 
    mvc_columns=@mvc_columns 
    RefreshColumns=@RefreshColumns 
></MVCTableColumnManager>

<MVCTable 
    TItem=@SharedTestData 
    data=@data 
    visible_col_names=@visible_mvc_columns
></MVCTable>

@code {
    //prepare RenderFragments to display the data in every column of every row
    public static RenderFragment<SharedTestData> ValColId = 
        context  => 
            __builder =>{ <td scope="row" ><b>@context.id</b> </td>};
    public static RenderFragment<SharedTestData> ValColName = 
        context  => 
            __builder =>{ <td  >@context.name </td>};
    public static RenderFragment<SharedTestData> ValColDescription = 
        context  => 
            __builder =>{ <td style="overflow: hidden; text-overflow: ellipsis; white-space: nowrap" >@context.description </td>};
    public static RenderFragment<SharedTestData> ValColExecutedOn = 
        context  => 
            __builder =>{ <td  >@context.executed_on.ToString("yyyy-MM-dd") </td>};
    public static RenderFragment<SharedTestData> ValColExecutedBy = 
        context  => 
            __builder =>{ <td  >@context.executed_by </td>};
    public static RenderFragment<SharedTestData> ValColSuccess = 
        context  => 
                __builder =>{ <td class="text-center" >@( @context.success ?"☑":"☐")</td>};
    public static RenderFragment<SharedTestData> ValColErrorCF = 
        context  => 
            __builder =>{ <td class="text-end" >@context.error_CF </td>
    };
    //define all columns to be displayed in the table
    public MVCColumnCtrl<SharedTestData>[] mvc_columns = new MVCColumnCtrl<SharedTestData>[]
    {
        new MVCColumnCtrl<SharedTestData>("id" ,  ValColId , "#" ),
        new MVCColumnCtrl<SharedTestData>("name" , ValColName , "name" ),
        new MVCColumnCtrl<SharedTestData>("description" ,ValColDescription ) {visible = true},
        new MVCColumnCtrl<SharedTestData>("executed_on" ,ValColExecutedOn ){visible = true},
        new MVCColumnCtrl<SharedTestData>("executed_by" ,ValColExecutedBy ){visible = true},
        new MVCColumnCtrl<SharedTestData>("success" ,ValColSuccess ){visible = true},
        new MVCColumnCtrl<SharedTestData>("error_CF" ,ValColErrorCF ){visible = true}
    };
    //visible_mvc_columns will only contain mvc_columns where the visible property is set to true in the RefreshColumns method
    public static MVCColumnCtrl<SharedTestData>[] visible_mvc_columns = new MVCColumnCtrl<SharedTestData>[]
        {
        new MVCColumnCtrl<SharedTestData>("id" ,  ValColId , "#" ),
        new MVCColumnCtrl<SharedTestData>("name" , ValColName , "name" )
    };
    /// <summary>run at OnInitialized and pass to MVCTableColumnManager</summary>
    public void RefreshColumns()
    {
        visible_mvc_columns = mvc_columns.Where(col => col.visible == true).ToArray();
        StateHasChanged();
    }
    /// <summary>
    /// sample dataset
    /// </summary>
    public SharedTestData[] data = new SharedTestData[]
    {
        new SharedTestData(){ id="ET220301", name="Efficiency", description="test for efficiency of program", executed_by="RX1",executed_on= DateTime.Now, error_CF= 0.31M, success=true },
        new SharedTestData(){ id="ET220302", name="Concurrency", description="test for ability to manage resources", executed_by="RX1",executed_on= DateTime.Now, error_CF= 0.21M, success=false },
        new SharedTestData(){ id="ET220303", name="Effability", description="test for ability eff with system", executed_by="RX1",executed_on= DateTime.Now, error_CF= 0M, success=false },
        new SharedTestData(){ id="ET220304", name="Edibility", description="test for edibility", executed_by="RX1",executed_on= DateTime.Now, error_CF= 0.8M, success=true },
        new SharedTestData(){ id="ET220305", name="Efficiency", description="test for efficiency of program", executed_by="RX2",executed_on= DateTime.Now, error_CF= 0.31M, success=true },
        new SharedTestData(){ id="ET220306", name="Concurrency", description="test for ability to manage resources", executed_by="RX2",executed_on= DateTime.Now, error_CF= 0.21M, success=false },
        new SharedTestData(){ id="ET220307", name="Effability", description="test for ability to eff with system", executed_by="RX2",executed_on= DateTime.Now, error_CF= 0M, success=false },
        new SharedTestData(){ id="ET220308", name="Edibility", description="test for edibility", executed_by="RX2",executed_on= DateTime.Now, error_CF= 0.8M, success=true }
    };
    protected override void OnInitialized()
    {
        RefreshColumns();
    }
}

```