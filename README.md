import { useState, useEffect, useMemo } from "react";
import { useLocation, useNavigate } from "react-router-dom";
import {
  modalSizes,
  modalTypes,
} from "@components/common/modal/modalConstants";
import Modal from "@components/common/modal";
import { useDispatch, useSelector } from "react-redux";
import {
  addLoader,
  removeLoader,
  updateStatusCurrency,
  DeleteStatusCurrency,
} from "@redux/commonSlice";
import AlertDialog from "@modules-(onboarding)/signIn/components/AlertDialog";
import { request } from "@utils";
import { updateCurrencyStatus } from "@services/currency";
import RowsPerPage from "@components/common/RowsPerPage";
import ViewFilter from "./ViewFilter";
import { getDefaultValue } from "../../../utils/getDefaultValues";
import { Button, Grid } from "@mui/material";
import MultiSelectDropdown from "../../../common/MultiselectDropdown.js/MultiSelect";
import ToggleInput from "../../../common/ToggleSwitch/ToggleInput";
import CustomTable from "../../../common/TableMUI/Table";
import { currencyList } from "../../../services-(onboarding)/currency";
import HtmlTooltip from "../../../common/CustomTooltip";
import EditNoteIcon from "@mui/icons-material/EditNote";
import DeleteOutlineOutlinedIcon from "@mui/icons-material/DeleteOutlineOutlined";
import VisibilityIcon from "@mui/icons-material/Visibility";
const pivoteSelect = [
  // { label: 'All', value: "All" },
  { label: "Yes", value: true },
  { label: "No", value: false },
];
const Index = (props) => {
  const { access = {} } = props;
  const navigate = useNavigate();
  const { state } = useLocation();
  const [offsetConst] = useState(state?.offset == null ? 0 : state?.offset);
  const [ismodalVisible, setismodalVisible] = useState(false);
  const [postsPerPage, setPostsPerPage] = useState(10);
  const [offset, setOffset] = useState(0);
  const [ismodaDelete, setismodaDelete] = useState("");
  const [item, setitem] = useState("");
  // const userId = useSelector((state) => state.userReducer.userId);
  // const [status, setStatus] = useState("");
  const [currencyLevelDropdown, setCurrencyLevelDrop] = useState([]);
  const [currency, setcurrency] = useState([]);
  const [currencySelect, setCurrencySelect] = useState([]);
  const [currencyTypeDropdown, setCurrencyTypeDrop] = useState([]);
  const [currencyTypeFilter, setCurrencyTypeFilter] = useState(null);
  const [currencySignDigitDropdown, setCurrencySignDigitDrop] = useState([]);
  const [currencyForexSignDigitDropdown, setCurrencyForexSignDigitDrop] = useState([]);
  const [currencySignDigitFilter, setCurrencySignDigitFilter] = useState([]);
  const { masterValueList, defaultValueList } = useSelector(
    (state) => state.commonReducer
  );
  const [masterValue, setmasterValue] = useState();
  const dispatch = useDispatch();
  const [popUp, setPopUp] = useState(false);
  const [filters, setFilter] = useState({
    type: [],
    digits: [],
    currency: [],
    level: [],
    pivot: [],
    status: [],
    forexDigits:[]
  });
  const openViewFilter = () => {
    setPopUp(true);
  };
  const getMasterValue = async () => {
    dispatch(addLoader("ACTION"));
    if (masterValueList.length) {
      let filterLevel = masterValueList
        .filter((x) => x.master_value_type === "CURRENCY_LEVEL")
        ?.sort(
          (b, a) => b.master_value_display_order - a.master_value_display_order
        )
        .map((item) => {
          const container = {};
          (container.label = item.master_value_name),
            (container.value = item.master_value_value);
          return container;
        });
      let filterType = masterValueList
        .filter((x) => x.master_value_type === "CURRENCY_TYPE")
        ?.sort(
          (b, a) => b.master_value_display_order - a.master_value_display_order
        )
        .map((item) => {
          const container = {};
          (container.label = item.master_value_name),
            (container.value = item.master_value_value);
          return container;
        });
      let filterSigndigit = masterValueList
        .filter((x) => x.master_value_type === "CURRENCY_SIGNIFICANT_DIGITS")
        ?.sort(
          (b, a) => b.master_value_display_order - a.master_value_display_order
        )
        .map((item) => {
          const container = {};
          (container.label = item.master_value_name),
            (container.value = item.master_value_value);
          return container;
        });
      let filterForexSigndigit = masterValueList
        .filter((x) => x.master_value_type === "FOREX_SIGNIFICANT_DIGITS")
        ?.sort(
          (b, a) => b.master_value_display_order - a.master_value_display_order
        )
        .map((item) => {
          const container = {};
          (container.label = item.master_value_name),
            (container.value = item.master_value_value);
          return container;
        });
      const statusMaster = masterValueList
        ?.filter((item) => item.master_value_type === "STATUS")
        .sort(
          (b, a) => b.master_value_display_order - a.master_value_display_order
        )
        .map((item) => ({
          value: item.master_value_value,
          label: item.master_value_name,
          name: "status",
          ...item,
        }))
        .filter((item) => item.master_value_name !== "Deleted");
      console.log("statusMaster", statusMaster);
      setmasterValue(statusMaster);
      setCurrencyTypeDrop(filterType);
      setCurrencySignDigitDrop(filterSigndigit);
      setCurrencyForexSignDigitDrop(filterForexSigndigit);
      setCurrencyLevelDrop(filterLevel);
    }
    dispatch(removeLoader("ACTION"));
  };
  function getMasterTextByValue(value, type) {
    if (masterValueList?.length) {
      const dataType = masterValueList?.filter(
        (item) =>
          value == item.master_value_value && type == item.master_value_type
      );
      if (dataType.length > 0) {
        return dataType[0].master_value_name;
      } else {
        return "NA";
      }
    }
    return "NA";
  }
const getCurrencies = async () => {
    try {
      dispatch(addLoader("CURRENCY_ACTION"));
      let response = await request({ api: currencyList });
      let data = response?.data?.data;
      const selectedCurrency = data.map((item) => {
        const container = { ...item };
        container.label = `${item.currency_abbreviation} (${item.currency_code})`;
        container.value = item.currency_code;
        container.currencycode = item.currency_code;
        container.currency_numeric_code = item.currency_numeric_code;
        container.currencyname = item.currency_abbreviation;
        container.currencylevel = item.currency_level;
        container.type = item.currency_type;
        container.roundoff = item.round_off_rule;
        container.fx_round_off_rule = item.fx_round_off_rule;
        container.ispivot = item.isPivot;
        container.status = item.status;
        return container;
      });
      setCurrencySelect(
        selectedCurrency.map((item, index) => ({ ...item, srNo: index + 1 }))
      );
      setcurrency(
        data
          .map((val) => ({
            label: `${val.currency_abbreviation} (${val.currency_code})`,
            value: val.currency_code,
            ...val,
          }))
          .map((item, index) => ({ ...item, srNo: index + 1 }))
      );
      if (state?.offset && state?.ppp) {
        setOffset(parseInt(state?.offset));
        setPostsPerPage(state.ppp);
      }
      if (state?.filter !== undefined) {
        setFilter(state?.filter);
      }
    } catch (error) {
      console.log(error);
    } finally {
      dispatch(removeLoader("CURRENCY_ACTION"));
    }
  };
  const updateStatus = async () => {
    try {
      dispatch(addLoader("ACTION"));
      const datas = {
        id: item.currency_code,
        // userId: userId,
        status: ismodaDelete?.toUpperCase(),
      };
      dispatch(addLoader("ACTION"));
      const response = await request({
        api: updateCurrencyStatus,
        params: datas,
      });
      if (response.status == 200 || response.status == 201) {
        dispatch(removeLoader("ACTION"));
        setismodalVisible({ status: true, message: response?.data?.message });
        // setismodalVisible(true);
        if (ismodaDelete?.toLowerCase() === "deleted") {
          dispatch(DeleteStatusCurrency({ id: item.currency_code }));
        } else {
          dispatch(
            updateStatusCurrency({
              id: item.currency_code,
              status: ismodaDelete?.toUpperCase(),
            })
          );
        }
        // getCurrencies()
      } else {
        dispatch(removeLoader("ACTION"));
      }
    } catch (error) {
      dispatch(removeLoader("ACTION"));
    }
  };
  const handleFilterSubmit = () => {
    // filterDataFun(check);
    setPopUp(false);
  };
  useEffect(() => {
    dispatch(addLoader("ACTION"));
    getCurrencies();
    getMasterValue();
    dispatch(removeLoader("ACTION"));
  }, [ismodalVisible, masterValueList, currencyList]);
  useEffect(() => {
    if (
      filters.currency.length > 0 ||
      filters.level.length > 0 ||
      filters.pivot.length > 0 ||
      filters?.status?.length > 0 ||
      filters?.digits?.length > 0 ||
      filters?.type?.length > 0
    ) {
      // filterDataFun();
    } else {
      // filterDataFun(true);
      setOffset(offsetConst);
    }
  }, [filters, ismodalVisible]);
  const columns = useMemo(
    () => [
      {
        field: "srNo",
        headerName: "Sl No",
        align: "center",
        width: "10px",
        renderCell: ({ row }) => {
          return <div>{row.srNo}</div>;
        },
      },
      {
        field: "currencycode",
        headerName: "Code",
        sortable: false,
        width: "100px",
      },
      {
        field: "name",
        headerName: "Name",
        sortable: false,
        width: "250px",
        renderCell: ({ row }) => {
          return row.currencyname;
        },
      },
      {
        field: "label",
        headerName: "Level",
        sortable: false,
        flex: 0.5,
        width: "100px",
        renderCell: ({ row }) => {
          return getMasterTextByValue(row.currencylevel, "CURRENCY_LEVEL");
        },
      },
      {
        field: "type",
        headerName: "Type",
        sortable: false,
        flex: 0.5,
        width: "100px",
        renderCell: (e) => {
          return getMasterTextByValue(e.row.type, "CURRENCY_TYPE");
        },
      },
      {
        field: "significantdigit",
        headerName: "Transaction Significant Digits",
        sortable: false,
        flex: 0.5,
        width: "220px",
        renderCell: (e) => {
          return getMasterTextByValue(
            e.row.roundoff,
            "CURRENCY_SIGNIFICANT_DIGITS"
          );
        },
      },
      {
        field: "forexSignificantdigit",
        headerName: "Forex Significant Digits",
        sortable: false,
        flex: 0.5,
        width: "220px",
        renderCell: (e) => {
          return getMasterTextByValue(
            e.row.fx_round_off_rule,
            "FOREX_SIGNIFICANT_DIGITS"
          );
        },
      },
      {
        field: "ispivot",
        headerName: "is Pivot",
        sortable: false,
        textAlign: "center",
        width: "100px",
        renderCell: ({ row }) => {
          return row.ispivot === true ? "Yes" : "No";
        },
      },
      {
        field: "pivot_currency",
        headerName: "Pivot Currency",
        sortable: false,
        flex: 0.5,
        width: "150px",
      },
      {
        field: "status",
        headerName: "Status",
        align: "center",
        width: "100px",
        renderCell: ({ row }) => {
          if (row.status) {
            return row.status?.toLowerCase() == "suspend" ||
              row.status?.toLowerCase() == "deleted" ? (
              row.status.charAt(0).toUpperCase() +
                row.status.slice(1).toLowerCase()
            ) : (
              <ToggleInput
                value={row.status.toUpperCase() === "ACTIVE" ? true : false}
                // value={e.row.status}
                disabled={!access?.EDIT}
                align={"center"}
                // eslint-disable-next-line no-unused-vars
                onChange={(event) => {
                  setitem(row),
                    setismodaDelete(
                      row.status?.toUpperCase() == "ACTIVE"
                        ? "Inactive"
                        : "Active"
                    );
                }}
              />
            );
          }
        },
      },
      {
        field: "actions",
        headerName: "Actions",
        width: "100px",
        renderCell: ({ row: item }) => {
          return (
            <div className="d-flex justify-content-center  gap-4">
              {!!access.READ && (
                <div className="view">
                  <a
                    onClick={(ev) => {
                      ev.preventDefault();
                      navigate(
                        `/configuration/view/currency/${item.currency_code}`,
                        {
                          state: {
                            offset: offsetConst,
                            ppp: postsPerPage,
                            data: item,
                            filter: filters,
                          },
                        }
                      );
                    }}
                    style={{
                      color: "#405189",
                      cursor: "pointer",
                    }}
                  >
                    <HtmlTooltip title="View" size="small">
                      <VisibilityIcon sx={{ width: "20px" }} />
                    </HtmlTooltip>
                  </a>
                </div>
              )}
              {access.EDIT ? (
                <div className="edit">
                  <a
                    onClick={(ev) => {
                      ev.preventDefault();
                      navigate(
                        `/configuration/edit/currency/${item.currency_code}/`,
                        {
                          state: {
                            offset: offsetConst,
                            ppp: postsPerPage,
                            filter: filters,
                          },
                        }
                      );
                    }}
                    style={{
                      color: "#405189",
                      cursor: "pointer",
                    }}
                  >
                    {/* {configConstants.EDIT} */}
                    <HtmlTooltip title={"Edit"} size="small">
                      <EditNoteIcon sx={{ width: "20px" }} />
                    </HtmlTooltip>
                  </a>
                </div>
              ) : null}
              {access.DELETE ? (
                <div className="remove">
                  <a
                    onClick={() => (setitem(item), setismodaDelete("Deleted"))}
                    style={{
                      color: "#405189",
                      cursor: "pointer",
                    }}
                  >
                    {" "}
                    {/* {configConstants.DELETE} */}
                    <HtmlTooltip title={"Delete"} size="small">
                      <DeleteOutlineOutlinedIcon sx={{ width: "20px" }} />
                    </HtmlTooltip>
                  </a>
                </div>
              ) : null}
            </div>
          );
        },
      },
    ],
    [currencyList, access, offsetConst, postsPerPage, filters]
  );
  const rowData = useMemo(() => {
    return currencySelect.filter((item) => {
      let flag = true;
      if (flag && filters.currency?.length) {
        flag =
          filters.currency.findIndex(
            (f) => f.currency_code === item.currency_code
          ) >= 0;
      }
      if (flag && filters.level.length) {
        flag =
          flag &&
          filters.level.findIndex(
            (f) => String(f.value) === String(item.currency_level)
          ) >= 0;
      }
      if (flag && filters.pivot.length) {
        flag =
          flag && filters.pivot.findIndex((f) => f.value === item.isPivot) >= 0;
      }
      if (flag && filters?.status?.length) {
        // if (filters?.status?.length === 0) return true; // No region filter
        return filters?.status?.some(
          (filter) =>
            filter?.label?.toString()?.toUpperCase() ===
            item?.status?.toString()?.toUpperCase()
        );
      }
      if (flag && filters.digits.length) {
        return filters?.digits?.some(
          (filter) =>
            filter.value?.toString() === item.round_off_rule?.toString()
        );
      }
      if (flag && filters.forexDigits.length) {
        return filters?.forexDigits?.some(
          (filter) =>
            filter.value?.toString() === item.fx_round_off_rule?.toString()
        );
      }
      if (flag && filters.type.length) {
        return filters?.type?.some(
          (filter) =>
            filter.value?.toString() === item.currency_type?.toString()
        );
      }
      return flag;
    });
  }, [filters, currencySelect]);
return (
    <>
      <div className="container-fluid">
        <div className="col-lg-12">
          <h3 className="card-title mb-3 flex-grow-1 text-dark">
            Currency Configuration
          </h3>
          <div className="card">
            <div className="card-header align-items-center d-flex">
              <h4 className="text-white" style={{ marginBottom: "0px" }}>
                All Currencies
              </h4>
            </div>
            <div className="card" style={{ marginBottom: "0px" }}>
              <div className="row p-2 Onboard mb-2">
                <div className="col-sm-auto mt-2">
  
   <Grid container spacing={4}>
                    <Grid item xs={12} sm={6} md={4}>
                      <MultiSelectDropdown
                        label="Currency"
                        options={currency}
                        placeHolder="All"
                        selectedOption={filters?.currency}
                        onChange={(_event, value) => {
                          console.log("currency value", value);
                          setFilter({ ...filters, currency: value });
                        }}
                        multiple
                        checkBox={false}
                        zIndex="1100"
                        size="small"
                        sx={{ minWidth: "180px" }}
                      />
                    </Grid>
 <Grid item xs={12} sm={6} md={4}>
                      <MultiSelectDropdown
                        label="Level"
                        options={currencyLevelDropdown}
                        placeHolder="All"
                        selectedOption={filters.level}
                        onChange={(_event, value) => {
                          setFilter({ ...filters, level: value });
                        }}
                        multiple
                        checkBox={false}
                        zIndex="1100"
                        size="small"
                        sx={{ minWidth: "180px" }}
                      />
                    </Grid>
<Grid item xs={12} sm={6} md={4}>
<MultiSelectDropdown
                        label="Status"
                        placeHolder="All"
                        selectedOption={filters.status}
                        options={masterValue}
                        onChange={(_event, value) => {
                          console.log(value);
                          const selectedValues = value.map((item) => ({
                            label: item.label,
                            value: item.value,
                          }));
                          setFilter({ ...filters, status: selectedValues });
                        }}
                        multiple
                        checkBox={false}
                        zIndex={1100}
                        size="small"
                        sx={{ minWidth: "180px" }}
                      />
                    </Grid>
                  </Grid>
                </div>
                <div className="col-sm">
                  <div
                    className="d-flex justify-content-sm-end"
                    style={{ marginTop: "10px" }}
                  >
                    {access.CREATE ? (
                      <div>
                        <a href="/add/currency">
                          <Button
  variant="contained"
                            onClick={(ev) => {
                              ev.preventDefault();
                              // navigate("/configuration/add/currency");
                              navigate(
                                `/configuration/add/currency/${offset}/${postsPerPage}`,
                                {
                                  state: {
                                    offset: offsetConst,
                                    ppp: postsPerPage,
                                    filter: filters,
                                  },
                                }
                              ); //////////////////////////////
                            }}
                            // data-bs-target="#showModal"
                          >
                            Add Currency
                          </Button>
                        </a>
                      </div>
                    ) : null}
<RowsPerPage
                      setOffset={setOffset}
                      value={postsPerPage}
                      setPostsPerPage={setPostsPerPage}
                    />
                    <div>
                      <i
                        className="bx bx-filter-alt"
                        style={{ cursor: "pointer", margin: "10px" }}
                        onClick={openViewFilter}
                      />
                    </div>
                  </div>
                </div>
              </div>
  <CustomTable
                columns={columns}
                rows={rowData}
                getRowId={(row) => {
                  return `${row.currencycode}-${row.currency_numeric_code}`;
                }}
                totalItemsCount={rowData.length}
                page={offset}
                numberOfItemsPerPage={postsPerPage}
                onChangePage={(e) => {
                  setOffset((e - 1) * postsPerPage);
                }}
              />
            </div>
          </div>
        </div>
      </div>
      {popUp && (
        <ViewFilter
          isModalVisible={popUp}
          setModalVisible={setPopUp}
          currencyTypeDropdown={currencyTypeDropdown}
          currencyTypeFilter={currencyTypeFilter}
          setCurrencyTypeFilter={setCurrencyTypeFilter}
          currencySignDigitDropdown={currencySignDigitDropdown}
          currencyForexSignDigitDropdown={currencyForexSignDigitDropdown}
          currencySignDigitFilter={currencySignDigitFilter}
          filter={filters}
          setFilter={setFilter}
          pivoteSelect={pivoteSelect}
          setCurrencySignDigitFilter={setCurrencySignDigitFilter}
          onSubmit={handleFilterSubmit}
        />
      )}
      {ismodalVisible?.status ? (
        <Modal
          Type={modalTypes.STATUS}
          ismodalVisible={ismodalVisible?.status}
          onClick={(ev) => {
            ev.preventDefault();
            // setismodalVisible(false);
            setismodalVisible({ status: false, message: "" });
          }}
        >
          <h4 className="mb-3 mt-4">
            {/* {`Currency ${status}d Successfull !`} */}
            {ismodalVisible?.message}
          </h4>
        </Modal>
      ) : null}
      {ismodaDelete ? (
        <Modal
          Type={modalTypes.CONFIRM}
          ismodalVisible={ismodaDelete}
          size={modalSizes.LARGE}
          onClick={(ev) => {
            ev.preventDefault();
            updateStatus();
            setismodaDelete("");
          }}
          onClickBack={(ev) => {
            ev.preventDefault();
            setismodaDelete("");
          }}
        >
          <h2
            className="mb-3 mt-4 modal-header"
            style={{ display: "flex", justifyContent: "center" }}
          >
            {/* {`Do you want to ${ismodaDelete} Currency?`} */}
            {ismodaDelete?.toLowerCase() == "active"
              ? getDefaultValue(
                  "CURRENCY_STATUS_ENABLE_CONFIRMATION_MESSAGE",
                  defaultValueList
                ) || "Are you sure?"
              : getDefaultValue(
                  "CURRENCY_STATUS_DISABLE_CONFIRMATION_MESSAGE",
                  defaultValueList
                ) || "Are you sure?"}
          </h2>
        </Modal>
      ) : null}
      <AlertDialog />
    </>
  );
};
export default Index;
import { useState, useEffect, useMemo } from "react";
import { modalTypes } from "@components/common/modal/modalConstants";
import Modal from "@components/common/modal";
import { useDispatch, useSelector } from "react-redux";
import { addLoader, removeLoader, setCurrencyList } from "@redux/commonSlice";
import { useLocation, useNavigate, useParams } from "react-router-dom";
import AlertDialog from "@modules-(onboarding)/signIn/components/AlertDialog";
import { request } from "@utils";
import { addCurrency, editCurrency, getCurrencybyId } from "@services/currency";
import { CurrencyValidation } from "@utils/validation/CurrencyValidation";
import { ValidateCountry } from "../country/components/Validation";
import TextFieldComponent from "../../../common/TextField/TextFieldComponent";
import Dropdown from "../../../common/Dropdown/Dropdown";
import Button from "../../../common/Button/CommonButton";
import { Box, Grid } from "@mui/material";
import ToggleInput from "../../../common/ToggleSwitch/ToggleInput";
import MultiSelectDropdown from "../../../common/MultiselectDropdown.js/MultiSelect";
const AddCurrency = () => {
  const { state } = useLocation();
  const { currencyCode } = useParams();
  const [ismodalVisible, setismodalVisible] = useState({
    status: false,
    message: "",
  });
  const [masterValue, setmasterValue] = useState();
  const [pivotcurrList, setPivotList] = useState([]);
  // const [isSubmitDisabled,setIsSubmitDisabled]=useState(true)
  // const userId = useSelector((state) => state.userReducer.userId);
  const { masterValueList, currencyList } = useSelector(
    (state) => state.commonReducer
  );
  //const [currencyName,setcurrencyName] = useState("")
  // console.log('stateee',state);
  const [data, setData] = useState({
    pivot: false,
    status: "",
    name: "",
    code: "",
    significant: "",
    fx_round_off_rule: "",
    level: "",
    type: currencyCode ? "" : 500,
    pivot_currency: "",
    max_percentage_fee_threshold: "",
    max_fixed_fee_threshold: "",
  });
  const [formErrors, setFormErrors] = useState({
    pivot: false,
    status: "",
    name: "",
    code: "",
    significant: "",
    fx_round_off_rule: "",
    level: "",
    type: "",
    pivot_currency: "",
    max_percentage_fee_threshold: "",
    max_fixed_fee_threshold: "",
  });
  const isViewMode = window.location.pathname.includes("/view/");
  const handleChange = (ev) => {
    if (!isViewMode) {
      const { name, value } = ev.target;
      setData({
        ...data,
        [name]: value,
      });
      setFormErrors({
        ...formErrors,
        [name]: CurrencyValidation(name, name, value),
      });
    }
  };
  const significantChange = (ev, dataChange, nameTarget) => {
    const name = nameTarget || ev.target.name;
    const value = dataChange?.value || ev.target.value;
    setData({
      ...data,
      [name]: value,
    });
    setFormErrors({
      ...formErrors,
      [name]: CurrencyValidation(name, name, value),
    });
  };
  const isFormValid = () => {
    const temp = {};
    let isFormValid = true;
    // console.log("data", data);
    Object.keys(data).forEach((field) => {
      const status = CurrencyValidation(field, field, data[field]);
      temp[field] = status;
      if (status) isFormValid = false;
    });
    setFormErrors(temp);
    return isFormValid;
  };
  const handleCurrencyChange = (e) => {
    const { value, name } = e.target;
    const nameValid = new RegExp("^[a-zA-Z ]*$");
    if (nameValid.test(value)) {
      let updatedValue = {
        [name]: value,
      };
      console.log({ ...data, ...updatedValue });
      setData({ ...data, ...updatedValue });
      // CurrencyValidation(name,name,value)
      setFormErrors({
        ...formErrors,
        [name]: CurrencyValidation(name, name, value),
      });
    }
  };
  const dispatch = useDispatch();
  const navigate = useNavigate();
  useEffect(() => {
    getMaterValue();
  }, []);
  const getMaterValue = async () => {
    if (masterValueList.length) setmasterValue(masterValueList);
  };
  // console.log("masterValueList",masterValueList);
  const SaveCurrency = async () => {
    // console.log(isFormValid())
    if (!isFormValid()) {
      return false;
    }
    dispatch(addLoader("ACTION"));
    let datas = {
      currency_code: data.code,
      // userId: userId,
      status: data.status,
      fx_round_off_rule: data.fx_round_off_rule,
      currency_abbreviation: data.name,
      round_off_rule: data.significant,
      isPivot: data.pivot,
      currency_type: data.type,
      currency_level: data.level,
      pivot_currency: data?.pivot_currency.map((x) => x.value).join(","),
      max_percentage_fee_threshold: data.max_percentage_fee_threshold,
      max_fixed_fee_threshold: data.max_fixed_fee_threshold,
    };
    // console.log(datas)
    const response = await request({ api: addCurrency, payload: datas });
    if (response.status == 201) {
      dispatch(removeLoader("ACTION"));
      dispatch(setCurrencyList(dispatch));
      setismodalVisible({ status: true, message: response?.data?.message });
    } else {
      dispatch(removeLoader("ACTION"));
    }
  };
  const geCurrencyById = async () => {
    dispatch(addLoader("ACTION"));
    const response = await request({
      api: getCurrencybyId,
      params: { currencyCode: currencyCode },
    });
    console.log("currency datas by id", response.data.data[0]);
    if (response.status === 200) {
      const tempData = response?.data?.data[0] || {};
      setData({
        pivot: tempData?.isPivot,
        status: tempData?.status,
        name: tempData?.currency_abbreviation,
        code: tempData?.currency_code,
        significant: tempData?.round_off_rule,
        level: tempData?.currency_level,
        fx_round_off_rule: tempData?.fx_round_off_rule,
        pivot_currency: currencyList
          ?.filter((item) => item.status.toLowerCase() === "active")
          ?.map((item) => {
            return {
              label: `${item.currency_abbreviation}(${item.currency_code})`,
              value: item.currency_code,
            };
          })
          .filter((x) =>
            tempData?.pivot_currency?.split(",")?.some((y) => y === x.value)
          ),
        type: tempData?.currency_type,
        max_percentage_fee_threshold: tempData?.max_percentage_fee_threshold,
        max_fixed_fee_threshold: tempData?.max_fixed_fee_threshold,
      });
      dispatch(removeLoader("ACTION"));
    }
  };
  useEffect(() => {
    getCurrencies();
  }, []);
  useEffect(() => {
    if (currencyCode) {
      geCurrencyById();
    }
  }, [currencyCode]);
  const updateCurrency = async () => {
    // console.log("data", data);
    if (!isFormValid()) {
      return false;
    }
    dispatch(addLoader("ACTION"));
    let datas = {
      // userId: userId,
      status: data?.status,
      currency_abbreviation: data?.name,
      round_off_rule: data?.significant,
      isPivot: data?.pivot,
      fx_round_off_rule: data.fx_round_off_rule,
      currency_type: data?.type,
      currency_level: data?.level,
      pivot_currency: data?.pivot_currency?.map((x) => x.value).join(","),
      max_percentage_fee_threshold: data?.max_percentage_fee_threshold,
      max_fixed_fee_threshold: data?.max_fixed_fee_threshold,
    };
    const response = await request({
      api: editCurrency,
      params: { code: currencyCode },
      payload: datas,
    });
    if (response.status == 200) {
      dispatch(removeLoader("ACTION"));
      dispatch(setCurrencyList(dispatch));
      setismodalVisible({ status: true, message: response?.data?.message });
    } else {
      dispatch(removeLoader("ACTION"));
    }
  };
  const getCurrencies = async () => {
    if (currencyList.length) {
      const activeCurrencies = currencyList.filter(
        (item) => item.status.toLowerCase() === "active"
      );
      const pivotCurrency = activeCurrencies
        .filter((item) => item.isPivot == true)
        .map((item) => {
          const container = {};
          container.label = `${item?.currency_abbreviation} (${item.currency_code})`;
          container.value = item.currency_code;
          return container;
        });
      const normalCurrency = activeCurrencies
        .filter((item) => item.isPivot == false)
        .map((item) => {
          const container = {};
          container.label = `${item?.currency_abbreviation} (${item.currency_code})`;
          container.value = item.currency_code;
          return container;
        });
      const pivot = [
        {
          label: "-----------------Pivot Currency---------------------",
          value: "",
          isDisabled: true,
        },
      ];
      const normal = [
        {
          label: "-----------------Normal Currency--------------------",
          value: "",
          isDisabled: true,
        },
      ];
      const pivotCurrencyData = [
        ...pivot,
        ...pivotCurrency,
        ...normal,
        ...normalCurrency,
      ];
      setPivotList(pivotCurrencyData);
      dispatch(removeLoader("ACTION"));
    }
  };
  // console.log("Datra", data)
  useEffect(() => {
    getCurrencies();
  }, []);
  const findSignificant = () => {
    const value = masterValueList
      ?.filter(
        (item) => item.master_value_type === "CURRENCY_SIGNIFICANT_DIGITS"
      )
      .sort(
        (a, b) => a.master_value_display_order - b.master_value_display_order
      )
      .find((item) =>
        data?.significant
          ? item.master_value_value?.includes(data?.significant)
          : null
      );
    return value
      ? {
          label: value.master_value_name,
          value: value.master_value_value,
        }
      : null;
  };
  const findForexSignificant = () => {
    const value = masterValueList
      ?.filter((item) => item.master_value_type === "FOREX_SIGNIFICANT_DIGITS")
      .sort(
        (a, b) => a.master_value_display_order - b.master_value_display_order
      )
      .find((item) => data?.fx_round_off_rule == item.master_value_value);
    return value
      ? {
          label: value.master_value_name,
          value: value.master_value_value,
        }
      : null;
  };
  const findLevel = () => {
    const value = masterValueList
      ?.filter((item) => item.master_value_type === "CURRENCY_LEVEL")
      .sort(
        (a, b) => a.master_value_display_order - b.master_value_display_order
      )
      .find((item) =>
        data?.level ? item.master_value_value?.includes(data?.level) : null
      );
    // console.log("Level", value, data?.level);
    return value
      ? {
          label: value.master_value_name,
          value: value.master_value_value,
        }
      : null;
  };
  const findType = () => {
    const value = masterValueList
      ?.filter((item) => item.master_value_type === "CURRENCY_TYPE")
      .sort(
        (a, b) => a.master_value_display_order - b.master_value_display_order
      )
      .find((item) => item.master_value_value?.includes(data?.type));
    // console.log("type", value);
    return value
      ? {
          label: value.master_value_name,
          value: value.master_value_value,
        }
      : null;
  };
  // console.log("significant digits", data);
  const getValue = useMemo(() => {
    const value = masterValue
      ?.filter((item) => item.master_value_type === "STATUS")
      .find(
        (item) =>
          data?.status?.toLowerCase() === item.master_value_name?.toLowerCase()
      );
    return value
      ? {
          label: value.master_value_name,
          value: value.master_value_value,
          name: "status",
        }
      : null;
  }, [data.status]);
return (
    <>
      <a>
        <button
          type="button"
          className="btn add-btn d-flex p-0 p-1"
          // onClick={() => navigate(-1)}
          onClick={() => navigate(`/configuration/currency`, { state: state })}
        >
          <i className="bx bx-chevron-left" style={{ marginRight: "4px" }} />
          Back
        </button>
      </a>
      <div className="container-fluid">
        <div className="d-flex mt-2">
          <h4 className="card-title mb-0 flex-grow-1">
            {isViewMode ? "View Currency" : currencyCode ? "Edit Currency" : "Add New Currency"}
          </h4>
        </div>
        {masterValue !== "" ? (
          <div className="row mt-4">
            <div className="col-lg-12">
              <div className="card">
                <div
                  className="card-body-currency"
                  style={{ backgroundColor: "#F1F3F6" }}
                >
                  <h5 className="information mb-0 flex-grow-1">
                    Currency Information
                  </h5>
                  <form>
                    <div className="card-body">
                      <div className="live-preview ">
                        <Grid container spacing={3}>
  <Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <TextFieldComponent
                              name="name"
                              type="text"
                              placeholder="Currency Name *"
                              label="Currency Name"
                              height="48px"
                              size="normal"
                              maxLength={20}
                              minLength={3}
                              error={
                                formErrors.name && formErrors.name.trim() !== ""
                              }
                              helperText={formErrors.name}
                              onChange={handleCurrencyChange}
                              value={data.name}
                              disabled={isViewMode}
                            />
                          </Grid>
<Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <TextFieldComponent
                              name="code"
                              placeholder="Currency Code *"
                              height="48px"
                              size="normal"
                              label="Currency Code"
                              maxLength={3}
                              error={formErrors.code}
                              helperText={formErrors.code}
                              onChange={handleChange}
                              value={data.code}
                              disabled={isViewMode}
                            />
                          </Grid>
                          <Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <Dropdown
                              height="48px"
                              size="normal"
                              name="significant"
                              placeholder={"Transaction Significant Digits *"}
                              label={"Transaction Significant Digits *"}
                              selectedOption={findSignificant()}
                              value={data?.significant}
                              zIndex={"1100"}
                              options={masterValue
                                ?.filter(
                                  (item) =>
                                    item.master_value_type ===
                                    "CURRENCY_SIGNIFICANT_DIGITS"
                                )
                                .sort(
                                  (a, b) =>
                                    a.master_value_display_order -
                                    b.master_value_display_order
                                )
                                .map((item) => ({
                                  value: item.master_value_value,
                                  label: item.master_value_name,
                                  name: "significant",
                                  ...item,
                                }))}
                              onChange={(e, data) =>
                                significantChange(e, data, "significant")
                              }
                              error={formErrors.significant}
                              helperText={formErrors.significant}
                              disabled={isViewMode}
                            />
                          </Grid>
                          <Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <Dropdown
                              height="48px"
                              size="normal"
                              name="fx_round_off_rule"
                              placeholder={"Forex Significant Digits *"}
                              label={"Forex Significant Digits *"}
                              selectedOption={findForexSignificant()}
                              value={data?.fx_round_off_rule}
                              zIndex={"1100"}
                              options={masterValue
                                ?.filter(
                                  (item) =>
                                    item.master_value_type ===
                                    "FOREX_SIGNIFICANT_DIGITS"
                                )
                                .sort(
                                  (a, b) =>
                                    a.master_value_display_order -
                                    b.master_value_display_order
                                )
                                .map((item) => ({
                                  value: item.master_value_value,
                                  label: item.master_value_name,
                                  name: "fx_round_off_rule",
                                  ...item,
                                }))}
                              onChange={(e, data) =>
                                significantChange(e, data, "fx_round_off_rule")
                              }
                              error={formErrors.fx_round_off_rule}
                              helperText={formErrors.fx_round_off_rule}
                              disabled={isViewMode}
                            />
                          </Grid>
  <Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <Dropdown
                              height="48px"
                              name="level"
                              size="normal"
                              placeholder={"Currency Level *"}
                              label={"Currency Level *"}
                              zIndex={"1100"}
                              selectedOption={findLevel()}
                              value={data.level}
                              error={formErrors.level}
                              options={masterValue
                                ?.filter(
                                  (item) =>
                                    item.master_value_type === "CURRENCY_LEVEL"
                                )
                                .sort(
                                  (a, b) =>
                                    a.master_value_display_order -
                                    b.master_value_display_order
                                )
                                .map((item) => ({
                                  value: item.master_value_value,
                                  label: item.master_value_name,
                                  name: "level",
                                  ...item,
                                }))}
                              onChange={(e, data) =>
                                significantChange(e, data, "level")
                              }
                              helperText={formErrors.level}
                              disabled={isViewMode}
                            />
                          </Grid>
<Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <Dropdown
                              height="48px"
                              name="type"
                              size="normal"
                              placeholder={"Currency Type *"}
                              label={"Currency Type *"}
                              value={data.type}
                              selectedOption={findType()}
                              zIndex={"1100"}
                              error={formErrors.type}
                              options={masterValue
                                ?.filter(
                                  (item) =>
                                    item.master_value_type === "CURRENCY_TYPE"
                                )
                                .sort(
                                  (b, a) =>
                                    b.master_value_display_order -
                                    a.master_value_display_order
                                )
                                .map((item) => ({
                                  value: item.master_value_value,
                                  label: item.master_value_name,
                                  name: "type",
                                  ...item,
                                }))}
                              onChange={(e, data) =>
                                significantChange(e, data, "type")
                              }
                              helperText={formErrors.type}
                              disabled={isViewMode}
                            />
                          </Grid>
<Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <MultiSelectDropdown
                              showTooltip
                              height="48px"
                              chipWidth="100%"
                              name="level"
                              size="normal"
                              multiple={true}
                              //  placeholder={"Pivot Currency *"}
                              label={"Pivot Currency *"}
                              zIndex={"1100"}
                              value={data.pivot_currency}
                              selectedOption={data?.pivot_currency || []}
                              error={formErrors.pivot_currency}
                              options={pivotcurrList}
                              onChange={(event, val) => {
                                // const value=yx.value;
                                if (
                                  !val?.length ||
                                  val?.length < 12 ||
                                  data?.pivot_currency?.length < 12
                                ) {
                                  setData({
                                    ...data,
                                    pivot_currency: val,
                                  });
                                } else {
                                  setData({
                                    ...data,
                                    pivot_currency: [
                                      ...(data.pivot_currency || []),
                                    ],
                                  });
                                }
                                setFormErrors({
                                  ...formErrors,
                                  pivot_currency: ValidateCountry(
                                    "pivot_currency",
                                    val
                                  ),
                                });
                              }}
                              helperText={formErrors.pivot_currency}
                              disabled={isViewMode}
                            />
                          </Grid>
 <Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <TextFieldComponent
                              name="max_percentage_fee_threshold"
                              placeholder="Max Percentage Fee"
                              height="48px"
                              size="normal"
                              maxLength={6}
                              minLength={1}
                              label="Max Percentage Fee"
                              error={formErrors.max_percentage_fee_threshold}
                              helperText={
                                formErrors.max_percentage_fee_threshold
                              }
                              onChange={handleChange}
                              value={data.max_percentage_fee_threshold}
                              disabled={isViewMode}
                            />
                          </Grid>
<Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <TextFieldComponent
                              name="max_fixed_fee_threshold"
                              placeholder="Max Fixed Fee"
                              height="48px"
                              size="normal"
                              label="Max Fixed Fee"
                              maxLength={10}
                              minLength={1}
                              error={formErrors.max_fixed_fee_threshold}
                              helperText={formErrors.max_fixed_fee_threshold}
                              onChange={handleChange}
                              value={data.max_fixed_fee_threshold}
                              disabled={isViewMode}
                            />
                          </Grid>
  <Grid item xs={12} sm={6} md={4} lg={4} xl={4}>
                            <Dropdown
                              height="48px"
                              size="normal"
                              placeholder={"Currency Status *"}
                              label="Currency Status *"
                              options={masterValue
                                ?.filter(
                                  (item) => item.master_value_type === "STATUS"
                                )
                                .sort(
                                  (b, a) =>
                                    b.master_value_display_order -
                                    a.master_value_display_order
                                )
                                .map((item) => ({
                                  value: item.master_value_value,
                                  label: item.master_value_name,
                                  name: "status",
                                  ...item,
                                }))}
                              error={formErrors.status}
                              helperText={formErrors.status}
                              // selectedOption={getValue(masterData?.status, countryData?.status)}
                              selectedOption={getValue}
                              value={data?.status}
                              onChange={(event, val) => {
                                setData({
                                  ...data,
                                  status: val.master_value_name,
                                });
                                setFormErrors({
                                  ...formErrors,
                                  status: CurrencyValidation(
                                    "status",
                                    "status",
                                    val.master_value_name
                                  ),
                                });
                              }}
                              disabled={isViewMode}
                            />
                          </Grid>
                          <Grid item xs={12}>
                            <Box>
                              <ToggleInput
                                label=" Is it a Pivot Currency ?"
                                value={data.pivot}
                                disabled={isViewMode}
                                onChange={(isChecked) => {
                                  setData({
                                    ...data,
                                    pivot: isChecked,
                                  });
                                }}
                              />
                            </Box>
                          </Grid>
                        </Grid>
                        <Box
                          sx={{ display: "flex", justifyContent: "center" }}
                          mt={3}
                        >
                          <div className="m-3">
                            <Button
                              type="button"
                              variant="outlined"
                              sx={{ textTransform: "none" }}
                              onClick={() =>
                                navigate(`/configuration/currency`, {
                                  state: state,
                                })
                              }
                            >
                              Cancel
                            </Button>
                          </div>
                          {!isViewMode && (
                          <div className="m-3">
  <Button
                              width="150px"
                              height="38px"
                              onClick={
                                currencyCode ? updateCurrency : SaveCurrency
                              }
                              // disabled={isSubmitDisabled}
                            >
                              {!currencyCode
                                ? "Add Currency"
                                : "Confim & Proceed"}
                            </Button>
                          </div>
                          )}
                        </Box>
                      </div>
                    </div>
                  </form>
                </div>
              </div>
            </div>
          </div>
        ) : null}
      </div>
      {ismodalVisible?.status ? (
        <Modal
          Type={modalTypes.STATUS}
          ismodalVisible={ismodalVisible?.status}
          onClick={(ev) => {
            ev.preventDefault();
            setismodalVisible({ status: false, message: "" }),
              // navigate("/configuration/currency")
              navigate(-1, { state: state });
          }}
        >
          <h4 className="mb-3 mt-4">{ismodalVisible?.message}</h4>
        </Modal>
      ) : null}
      <AlertDialog />
    </>
  );
};
export default AddCurrency;
import { useState, useEffect, useMemo } from "react";
import { useDispatch } from "react-redux";
import { Grid } from "@mui/material";
import { addLoader, removeLoader } from "@redux/commonSlice";
import MultiSelectDropdown from "@/common/MultiselectDropdown.js/MultiSelect";
import RowsPerPage from "@components/common/RowsPerPage";
import CustomTable from "../../../common/TableMUI/Table";
import HtmlTooltip from "../../../common/CustomTooltip";
import EditNoteIcon from "@mui/icons-material/EditNote";
import DeleteOutlineOutlinedIcon from "@mui/icons-material/DeleteOutlineOutlined";
import Modal from "@components/common/modal";
import { modalSizes, modalTypes } from "@components/common/modal/modalConstants";

const ScheduleTransaction = (props) => {
    const { access = {EDIT: true, DELETE: true} } = props;
    const [postsPerPage, setPostsPerPage] = useState(10);
    const [offset, setOffset] = useState(0);
    const [schedules, setSchedules] = useState([]);
    const [deleteModal, setDeleteModal] = useState(false);
    const [selectedSchedule, setSelectedSchedule] = useState(null);
    const [statusModal, setStatusModal] = useState({ status: false, message: "" });

    const scheduleTypeOptions = [
        {label: "Hourly",value: "hourly"},
        {label: "Daily",value: "daily"},
        {label: "Weekly",value: "weekly"},
        {label: "Monthly",value: "monthly"},
    ];

    const [filters, setFilter] = useState({
        scheduleType: [],
    });

    const filteredSchedules = useMemo(() => {
        return schedules.filter((item) => {
            if (filters.scheduleType.length > 0) {
                return filters.scheduleType.some(filter => filter.value === item.schedule_type);
            }
            return true;
        });
    }, [filters, schedules]);

    const dispatch = useDispatch();

    const getAuthToken = async () => {
        try {
            const response = await fetch('/reports/tokenInitiation', {
                method: 'POST'
            });
            const data = await response.json();
            console.log("Response ", data);
            if (data.success) {
                return data.token;
            }
            throw new Error('Authentication failed');
        } catch (error) {
            console.error('Auth error:', error);
            return null;
        }
    };

    const getSchedules = async () => {
        try {
            dispatch(addLoader("SCHEDULE_ACTION"));
            const token = await getAuthToken();
            if (!token) {
                throw new Error('No auth token');
            }
           
            const response = await fetch('/reports/findScheduleByType/T', {
                method: 'GET',
                headers: {
                    'Authorization': `Bearer ${token}`,
                    'Content-Type': 'application/json'
                }
            });
           
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
           
            const result = await response.json();
            console.log("schedule response",result);
            console.log("Schedule data",result.data);
            if (result.msg === "Successful" && result.data) {
                setSchedules(result.data);
            }

        } catch (error) {
            console.error("Error fetching schedules:", error);
            setSchedules([]);
        } finally {
            dispatch(removeLoader("SCHEDULE_ACTION"));
        }
    };

    const handleDelete = async () => {
        if (!selectedSchedule) return;
        console.log("Schedule id to delete",selectedSchedule.id);
        setDeleteModal(false);
        setSelectedSchedule(null);
        setStatusModal({
            status: true,
            message: "Delete"
        });
        /* try {
            dispatch(addLoader("DELETE_ACTION"));
            const token = await getAuthToken();
            if (!token) {
                throw new Error('No auth token');
            }

            const response = await fetch(`/reports/deleteSchedule/${selectedSchedule.id}`, {
                method: 'DELETE',
                headers: {
                    'Authorization': `Bearer ${token}`,
                    'Content-Type': 'application/json'
                }
            });

            const result = await response.json();
           
            if (response.ok) {
                setStatusModal({
                    status: true,
                    message: result.msg || "Schedule deleted successfully"
                });
                getSchedules();
            } else {
                setStatusModal({
                    status: true,
                    message: "Failed to delete schedule"
                });
            }
        } catch (error) {
            console.error('Delete failed:', error);
            setStatusModal({
                status: true,
                message: "Error deleting schedule"
            });
        } finally {
            dispatch(removeLoader("DELETE_ACTION"));
            setDeleteModal(false);
            setSelectedSchedule(null);
        } */
    };

    const handleEdit = async (scheduleId) => {
        try {
            dispatch(addLoader("EDIT_ACTION"));
            const token = await getAuthToken();
            if (!token) {
                throw new Error('No auth token');
            }

            const response = await fetch(`/reports/findById/${scheduleId}`, {
                method: 'GET',
                headers: {
                    'Authorization': `Bearer ${token}`,
                    'Content-Type': 'application/json'
                }
            });

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }

            const result = await response.json();
           
            if (result.msg === "Successful" && result.data) {
                console.log("Schedule data for edit:", result.data);
            }
        } catch (error) {
            console.error("Error fetching schedule details:", error);
        } finally {
            dispatch(removeLoader("EDIT_ACTION"));
        }
    };

    useEffect(() => {
        getSchedules();
    }, []);

    const columns = useMemo(
        () => [
            {
                field: "id",
                headerName: "Sl No",
                align: "center",
                width: "70px",
                renderCell: ({ row }) => {
                    return <div>{row.id}</div>;
                },
            },
            {
                field: "schedule_name",
                headerName: "Schedule Name",
                align: "center",
                sortable: false,
                flex: 1,
                minWidth: "250px",
            },
            {
                field: "schedule_type",
                headerName: "Schedule Type",
                align: "center",
                sortable: false,
                width: "150px",
                renderCell: ({ row }) => {
                    return row.schedule_type?.charAt(0).toUpperCase() + row.schedule_type?.slice(1);
                },
            },
            {
                field: "actions",
                headerName: "Actions",
                align: "center",
                width: "120px",
                renderCell: ({ row }) => {
                    return (
                        <div className="d-flex justify-content-center gap-4">
                            {access.EDIT && (
                                <div className="edit">
                                    <a
                                        onClick={(ev) => {
                                            ev.preventDefault();
                                            handleEdit(row.id);
                                        }}
                                        style={{
                                            color: "#405189",
                                            cursor: "pointer",
                                        }}
                                    >
                                        <HtmlTooltip title="Edit" size="small">
                                            <EditNoteIcon sx={{ width: "20px" }} />
                                        </HtmlTooltip>
                                    </a>
                                </div>
                            )}
                            {access.DELETE && (
                                <div className="remove">
                                    <a
                                        onClick={(ev) => {
                                            ev.preventDefault();
                                            setSelectedSchedule(row);
                                            setDeleteModal(true);
                                        }}
                                        style={{
                                            color: "#405189",
                                            cursor: "pointer",
                                        }}
                                    >
                                        <HtmlTooltip title="Delete" size="small">
                                            <DeleteOutlineOutlinedIcon sx={{ width: "20px" }} />
                                        </HtmlTooltip>
                                    </a>
                                </div>
                            )}
                        </div>
                    );
                },
            },
        ],
        [access]
    );

    return (
        <>
            <div className="container-fluid">
                <div className="col-lg-12">
                    <h3 className="card-title mb-3 flex-grow-1 text-dark">
                        Schedule Transaction Report
                    </h3>
                    <div className="card">
                        <div className="card-header align-items-center d-flex">
                            <h4 className="text-white" style={{ marginBottom: "0px" }}>
                                All Schedules
                            </h4>
                        </div>
                        <div className="card" style={{ marginBottom: "0px" }}>
                            <div className="row p-2 Onboard mb-2">
                                <div className="col-sm-auto mt-2">
                                    <Grid container spacing={4}>
                                        <Grid item xs={12} sm={6} md={4}>
                                            <MultiSelectDropdown
                                                label="Schedule Type"
                                                options={scheduleTypeOptions}
                                                placeHolder="All"
                                                selectedOption={filters.scheduleType}
                                                onChange={(_event, value) => {
                                                    setFilter({ ...filters, scheduleType: value });
                                                }}
                                                multiple
                                                checkBox={false}
                                                zIndex="1100"
                                                size="small"
                                                sx={{ minWidth: "180px" }}
                                            />
                                        </Grid>
                                    </Grid>
                                </div>
                                <div className="col-sm">
                                    <div
                                        className="d-flex justify-content-sm-end"
                                        style={{ marginTop: "10px" }}
                                    >
                                        <RowsPerPage
                                            setOffset={setOffset}
                                            value={postsPerPage}
                                            setPostsPerPage={setPostsPerPage}
                                        />
                                    </div>
                                </div>
                            </div>
                            <CustomTable
                                columns={columns}
                                rows={filteredSchedules}
                                getRowId={(row) => row.id}
                                totalItemsCount={filteredSchedules.length}
                                page={offset}
                                numberOfItemsPerPage={postsPerPage}
                                onChangePage={(e) => {
                                    setOffset((e - 1) * postsPerPage);
                                }}
                            />
                        </div>
                    </div>
                </div>
            </div>

            {deleteModal && (
                <Modal
                    Type={modalTypes.CONFIRM}
                    ismodalVisible={deleteModal}
                    size={modalSizes.LARGE}
                    onClick={(ev) => {
                        ev.preventDefault();
                        handleDelete();
                    }}
                    onClickBack={(ev) => {
                        ev.preventDefault();
                        setDeleteModal(false);
                        setSelectedSchedule(null);
                    }}
                >
                    <h2
                        className="mb-3 mt-4 modal-header"
                        style={{ display: "flex", justifyContent: "center" }}
                    >
                        Are you sure you want to delete this schedule?
                    </h2>
                    {selectedSchedule && (
                        <p style={{ textAlign: "center" }}>
                            {selectedSchedule.schedule_name}
                        </p>
                    )}
                </Modal>
            )}

            {statusModal.status && (
                <Modal
                    Type={modalTypes.STATUS}
                    ismodalVisible={statusModal.status}
                    onClick={(ev) => {
                        ev.preventDefault();
                        setStatusModal({ status: false, message: "" });
                    }}
                >
                    <h4 className="mb-3 mt-4">
                        {statusModal.message}
                    </h4>
                </Modal>
            )}
        </>
    );
};

export default ScheduleTransaction;

 
