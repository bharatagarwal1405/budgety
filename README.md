# budgety
a web application used to represent monthly budget including income , expense and available budget.

//BUDGET CONTROLLER
var budgetController = (function() {
    
    var Expense = function(id, description, value) {
        this.id = id;
        this.description = description;
        this.value = value;
        this.percentage = -1
    };
    
    
    Expense.prototype.calcPercentage = function (totalIncome) {
       
        if(totalIncome > 0) {
            this.percentage = Math.round(this.value/totalIncome * 100);
        }else {
            this.percentage = -1;
        }
    };
    
    Expense.prototype.getPercentage = function() {
        return this.percentage;
    };
    
    var Income = function(id, description, value) {
        this.id = id;
        this.description = description;
        this.value = value
    };
    
    var calculateTotal = function(type) {
        var sum = 0;
        data.allItems[type].forEach(function(cur) {
           sum +=cur.value; 
        });
        data.totals[type] = sum;
    };
    
    var data = {
        allItems: {
            exp: [],
            inc: []
        },
        totals: {
            exp: 0,
            inc: 0
        },
        budget: 0,
        percentage: -1
    };
    
    return {
        addItem: function(type, des, val) {
            var newItem, ID;
            
            //[1,2,3,4,5], next id = 6
            //[1,2,4,6,8], next id = 9
            // ID = last ID + 1
            
            //create new id
            if(data.allItems[type].length > 0){
                 ID = data.allItems[type][data.allItems[type].length - 1].id + 1;
            } else {
                ID = 0;
            }
           
             
            // create new item based on 'inc' or 'exp' type
            if (type === 'exp') {
                newItem = new Expense(ID, des, val);
            }else if (type === 'inc') {
                newItem = new Income(ID, des, val);
            }
            // push it into our data structure
            data.allItems[type].push(newItem);
            
            // return new element
            return newItem;
        }, 
        
        deleteItem: function(type, id) {
            var ids, index;
            
            // id = 6
            // data.allitems[type][id]
            //ids = [1 2 3 6 8]
            //index = 3
    
            ids = data.allItems[type].map(function(current){
            return current.id;
            });
            
            index = ids.indexOf(id);
            
            if(index !== -1) {
                data.allItems[type].splice(index, 1);
                
            }
        },
        
        calculateBudget: function() {
            
            // calculate total income and expenses
            calculateTotal('exp');
            calculateTotal('inc');
            // calculate the budget: income - expenses
            data.budget = data.totals.inc - data.totals.exp;
            // calculate the percentage of income that we spent
            if(data.totals.inc > 0) {
                 data.percentage = Math.round((data.totals.exp / data.totals.inc) * 100);
            } else {
                percentage = -1;
            }
           
        },
        calculatePercentages: function() {
          /*
          a = 20
          b=10
          c= 40
          total income = 100
          a= 20/100 = 20%
          b= 10/100 = 10%
          c= 40/100 = 40%
          */
            data.allItems.exp.forEach(function(cur){
                cur.calcPercentage(data.totals.inc); 
            });
        },
        
        getPercentage: function() {
            var allPerc = data.allItems.exp.map(function(cur){
                return cur.getPercentage();
            });
            return allPerc;
        },
    
        getBudget: function(){
            return {
                budget: data.budget,
                totalInc: data.totals.inc,
                totalExp: data.totals.exp,
                percentage: data.percentage
            }
        }, 
        testing: function(){
            console.log(data);
        }
    };
})();

var Expense = function(id, description, value) {
        this.id = id;
        this.description = description;
        this.value = value;
    };

// UI CONTROLLER
var UIController = (function (){
    
    var DOMstrings = {
        inputType: '.add__type',
        inputDescription: '.add__description',
        inputValue: '.add__value',
        inputBtn: '.add__btn',
        incomeContainer: '.income__list',
        expensesContainer: '.expenses__list',
        budgetLabel: '.budget__value',
        incomeLabel: '.budget__income--value',
        expensesLabel: '.budget__expenses--value',
        percentageLabel: '.budget__expenses--percentage',
        container: '.container',
        expensesPercLabel: '.item__percentage',
        dateLabel: '.budget__title--month'
    };
    
     var formatNumber =  function(num, type) {
            var numSplit,int,dec, type;
            /*
            + or - for before number
            exactly 2 decimal points
            comma seperating the thousands 
            2300 -> +2,300.00
            200 -> +200.00
            */
            
            num = Math.abs(num);
            num = num.toFixed(2);
            
            numSplit = num.split('.');
            int = numSplit[0];
         
            if(int.length > 3) {
                int = int.substr(0,int.length - 3) + ',' + int.substr(int.length - 3, 3);
                
            }
         
            dec = numSplit[1];
         
            return (type === 'exp' ?  '-' : '+') + ' ' + int + '.' + dec;
        
        };
    
     var nodeListForEach = function(list, callback) {
                for(var i= 0 ; i< list.length; i++) {
                    callback(list[i], i);
                }
            };

    return {
        getInput: function() {
            return {
             type: document.querySelector(DOMstrings.inputType).value,
             description: document.querySelector(DOMstrings.inputDescription).value,
             value: parseFloat(document.querySelector(DOMstrings.inputValue).value)
            };
        },
        
        addListItem: function (obj, type) {
            var html, newHtml, element; 
            // create html string with placeholer text 
            if (type === 'inc') {
                element = DOMstrings.incomeContainer;
                 html = '<div class="item clearfix" id="inc-%id%"><div class="item__description">%description%</div><div class="right clearfix"><div class="item__value">%value%</div><div class="item__delete"><button class="item__delete--btn"><i class="ion-ios-close-outline"></i></button></div></div></div>';
            }else if (type === 'exp') {
                element = DOMstrings.expensesContainer;
                html = '<div class="item clearfix" id="exp-%id%"><div class="item__description">%description%</div><div class="right clearfix"><div class="item__value">%value%</div><div class="item__percentage">21%</div><div class="item__delete"><button class="item__delete--btn"><i class="ion-ios-close-outline"></i></button></div></div></div>';
            }
            // replace the placeholder text with actual text
            newHtml = html.replace('%id%' , obj.id);
            newHtml = newHtml.replace('%description%', obj.description);
            newHtml = newHtml.replace('%value%', formatNumber (obj.value, type));
            
            // insert html into dom 
            document.querySelector(element).insertAdjacentHTML('beforeend', newHtml);
        },
        
        deleteListItem: function (selectorID) {
            var el = document.getElementById(selectorID);
            el.parentNode.removeChild(el);  
        },
    
    
        clearFields: function() {
            var fields, fieldsArr;
            fields = document.querySelectorAll(DOMstrings.inputDescription + ',' + DOMstrings.inputValue);
            fieldsArr = Array.prototype.slice.call(fields);
            fieldsArr.forEach(function(current, index, array) {
                current.value = "";
            });
            fieldsArr[0].focus();
        },
        
        displayBudget: function(obj) {
            var type;
            obj.budget > 0 ? type = 'inc' : type = 'exp';
            
            document.querySelector(DOMstrings.budgetLabel).textContent = formatNumber(obj.budget, type);
            document.querySelector(DOMstrings.incomeLabel).textContent = formatNumber(obj.totalInc, 'inc');
            document.querySelector(DOMstrings.expensesLabel).textContent = formatNumber(obj.totalExp, 'exp');
            
            if(obj.percentage > 0 ) {
                document.querySelector(DOMstrings.percentageLabel).textContent = obj.percentage + '%';
            } else {
                document.querySelector(DOMstrings.percentageLabel).textContent = '--';
            }
        },
        
        displayPercentages: function (percentage) {
        
            var fields = document.querySelectorAll(DOMstrings.expensesPercLabel);
            
                nodeListForEach( fields ,function(current, index){
                    
                    if(percentage[index] > 0) {
                        current.textContent = percentage[index] + '%';
                    }else{
                        current.textContent = '---';
                    }
                });
            
            },
        
            displayMonth: function() {
                var now, month, months,year;
            
                now = new Date();
                months = ['january', 'feburary', 'march', 'april', 'may', 'june', 'july', 'augest', 'september', 'october', 'november', 'december'];
                month = now.getMonth(); 
                year = now.getFullYear();
                document.querySelector(DOMstrings.dateLabel).textContent = months[month] + ' ' +year;
        },
        
           changedType: function() {
               
               var fields = document.querySelectorAll(
                   DOMstrings.inputType + ',' + DOMstrings.inputDescription + ',' + DOMstrings.inputValue);
               
               nodeListForEach(fields, function(cur) {
                   cur.classList.toggle('red-focus');
               });
               
               document.querySelector(DOMstrings.inputBtn).classList.toggle('red');
               
           },
            getDOMstrings: function() {
                return DOMstrings;
        }
    }; 
})(); 


// GLOBAL APP CONTROLLER
var controller = (function (budgetCtrl, UICtrl){

    var setupEventListners = function() {
        var DOM = UICtrl.getDOMstrings();
    document.querySelector(DOM.inputBtn).addEventListener('click', ctrlAddItem);
        
    document.addEventListener('keypress', function(event) {
        if(event.keyCode === 13 || event.which === 13) {
            ctrlAddItem();
        }
    }); 
        document.querySelector(DOM.container).addEventListener('click', ctrlDeleteItem);
        document.querySelector(DOM.inputType).addEventListener('change', UICtrl.changedType);
    };
    
    
    var updateBudget = function() {
        // 1. calculate and update the budget
        budgetCtrl.calculateBudget();
        
        // 2. return budget
       var budget = budgetCtrl.getBudget();
       
        // 3. display the budget on the UI
       UICtrl.displayBudget(budget);

    };
        
    var updatePercentages = function () {
        
        //1. calculate and update percentages
        budgetCtrl.calculatePercentages();
        
        //2. read percentages from budget controller
        var percentages = budgetCtrl.getPercentage();
        
        //3. update the ui with new percentages
        UICtrl.displayPercentages(percentages);
        
    };   
        
    var ctrlAddItem = function () {
        
        var input, newItem;
        // 1. get the filed input data
     input = UICtrl.getInput();
        
        if(input.description !== "" && !isNaN(input.value) && input.value > 0) {
         // 2. add the item to the budget controller
        newItem = budgetCtrl.addItem(input.type, input.description, input.value);   
       
        // 3. add the item to the ui
        UICtrl.addListItem(newItem, input.type);
            
        // 4. clear fields
        UICtrl.clearFields();
        
        //5. update budget
        updateBudget();
            
        //6. update percentages
        updatePercentages();    
        }

    };
    
    var ctrlDeleteItem = function(event) {
        var itemID, splitID;
        itemID = event.target.parentNode.parentNode.parentNode.parentNode.id;
        
        if(itemID) {
            
            // inc-1
            splitID = itemID.split('-');
            type = splitID[0];
            ID = parseInt(splitID[1]);
            
            // 1.delete the item from the data structure
            budgetController.deleteItem(type, ID);
            // 2.delete the item from the ui
            UICtrl.deleteListItem(itemID);
            // 3.update and show the budget
            updateBudget();
            // 4. update percentages
            updatePercentages();
        } 
    };

    return {
        init: function(){
            console.log('application is started');
            UICtrl.displayMonth();
            UICtrl.displayBudget({
                budget: 0,
                totalInc: 0,
                totalExp: 0,
                percentage: -1
            });
            setupEventListners();
        }
    };
    
})(budgetController, UIController);

controller.init();
