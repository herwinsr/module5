import React, { useState, useEffect } from "react";
import {
  Dialog,
  DialogTitle,
  DialogContent,
  DialogContentText,
  DialogActions,
  TextField,
  Button,
  Box,
  Typography,
  Grid,
  CircularProgress,
  Alert,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
} from "@mui/material";
import { getName } from "country-list";
import { getCountryName } from "@modules-(pms)/utils/countryUtils";
import { Warning } from "@mui/icons-material";
import { Colors } from "@/theme/colors";

function titleCase(str) {
  if (!str) return "";
  return str
    .toLowerCase()
    .split(" ")
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
    .join(" ");
}

const CustomModal = ({
  isOpen,
  onClose,
  onSubmit,
  title,
  actionButtonText,
  isApprove,
  proposalId,
}) => {
  const [comments, setComments] = useState("");
  const [renegotiationDetails, setRenegotiationDetails] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [lowRateWarnings, setLowRateWarnings] = useState([]);
  const [hasCustomRates, setHasCustomRates] = useState(false);
  const [itemDetails, setItemDetails] = useState([]);
  const [initialComments, setInitialComments] = useState("");
  const [originalRenegotiationDetails, setOriginalRenegotiationDetails] =
    useState(null);

  const formatCase = (str) => {
    if (!str) return "";

    return str
      .toLowerCase()
      .replace(/_/g, " ") // Replace underscores with spaces
      .split(" ")
      .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
      .join(" ");
  };

  const [proposalDetails, setProposalDetails] = useState(null);
   useEffect(() => {
    const fetchRateWarnings = async () => {
      if (!isOpen || !actionButtonText === "Approve" || !renegotiationDetails) {
        return;
      }
      console.log("Fetching rate warnings with:", {
        proposalId: renegotiationDetails.proposal_id,
        itemId: renegotiationDetails.item_id,
        currentRateDetails: renegotiationDetails,
      });

      if (
        isOpen &&
        actionButtonText === "Approve" &&
        renegotiationDetails &&
        renegotiationDetails.proposal_id &&
        renegotiationDetails.item_id
      ) {
        try {
          console.log("Fetching rate warnings for:", {
            proposalId: renegotiationDetails.proposal_id,
            itemId: renegotiationDetails.item_id,
            currentRate: renegotiationDetails.custom_fee_flat,
            minimumRate: renegotiationDetails.minimum_fee_flat,
            currentRateType: renegotiationDetails.custom_fee_type,
            minimumRateType: renegotiationDetails.minimum_fee_type,
          });

          if (
            renegotiationDetails.custom_fee_type ===
            renegotiationDetails.minimum_fee_type
          ) {
            if (renegotiationDetails.custom_fee_type === "FLAT") {
              if (
                parseFloat(renegotiationDetails.custom_fee_flat) <
                parseFloat(renegotiationDetails.minimum_fee_flat)
              ) {
                setLowRateWarnings([
                  {
                    type: "below_minimum",
                    country_code: renegotiationDetails.country_code,
                    current_rate: renegotiationDetails.custom_fee_flat,
                    walkaway_rate: renegotiationDetails.minimum_fee_flat,
                  },
                ]);
              }
            }
            if (renegotiationDetails.custom_fee_type === "PERCENTAGE") {
              if (
                parseFloat(renegotiationDetails.custom_fee_percentage) <
                parseFloat(renegotiationDetails.minimum_fee_percentage)
              ) {
                setLowRateWarnings([
                  {
                    type: "below_minimum",
                    country_code: renegotiationDetails.country_code,
                    current_rate: renegotiationDetails.custom_fee_percentage,
                    walkaway_rate: renegotiationDetails.minimum_fee_percentage,
                  },
                ]);
              }
            }
          }
 } catch (error) {
          console.error("Error fetching rate warnings:", error);
          setLowRateWarnings([]);
        }
      }
    };

    fetchRateWarnings();
  }, [isOpen, actionButtonText, renegotiationDetails]);
  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(comments);
  };

  return (
    <Dialog open={isOpen} onClose={onClose} maxWidth="md" fullWidth>
      <DialogTitle bgcolor={Colors.tpBlue} color={Colors.light}>
        {title}
      </DialogTitle>
      <DialogContent>
        {actionButtonText === "Revoke" ? (
          <>
            <DialogContentText sx={{ mb: 2 }}>
              Are you sure you want to revoke this proposal?
            </DialogContentText>
            <TextField
              autoFocus
              margin="dense"
              // label="Reason for Recall"
              label="Reason for Revoke"
              fullWidth
              multiline
              rows={4}
              value={comments}
              onChange={(e) => setComments(e.target.value)}
              required
            />
          </>
        ) : actionButtonText === "Approve" && renegotiationDetails ? (
          <>
            {loading ? (
              <Box display="flex" justifyContent="center" my={2}>
                <CircularProgress />
              </Box>
            ) : error ? (
              <Typography color="error">{error}</Typography>
            ) : (
              renegotiationDetails && (
                <Box mb={3}>
                  {lowRateWarnings.length > 0 && (
                    <Alert severity="warning" sx={{ mb: 3 }}>
                      {lowRateWarnings.map((warning, index) => (
                        <Box key={index}>
                          {warning.type === "below_minimum" && (
                            <Typography variant="body2">
                              Requested rate for {getCountryName(warning.country_code)}{" "}
                              ({warning.country_code}) -
                              {renegotiationDetails.payout_method
                                ? ` ${titleCase(
                                    renegotiationDetails.payout_method
                                  )} `
                                : " "}
                              is lower than minimum/walkaway rate of USD{" "}
                              {warning.walkaway_rate}.
                            </Typography>
                          )}
                          {warning.type === "mixed" && (
                            <Typography variant="body2">
                              {warning.message}
                            </Typography>
                          )}
                        </Box>
                      ))}
                    </Alert>
                  )}

                  {initialComments && (
                    <Box sx={{ mb: 3 }}>
                      <Typography variant="h6" gutterBottom color="primary">
                        Initial Comments from Regional Director
                      </Typography>
                      <Paper
                        variant="outlined"
                        sx={{ p: 2, bgcolor: "grey.50" }}
                      >
                        <Typography variant="body1">
                          {initialComments}
                        </Typography>
                      </Paper>
                    </Box>
                  )}

                  {actionButtonText === "Approve" && hasCustomRates && (
                    <Alert severity="warning" sx={{ mb: 3 }}>
                      Some items are at custom rate. Please verify rate before
                      approving.
                    </Alert>
                  )}

                  <Grid container spacing={2}>
                    <Grid item xs={12}>
                      <Typography variant="h6" gutterBottom>
                        Renegotiation Request Details
                      </Typography>
                    </Grid>
                    <Grid item xs={6}>
                      <Typography variant="subtitle2">Country:</Typography>
                      <Typography>
                        {originalRenegotiationDetails?.country_code
                          ? `${getCountryName(
                              originalRenegotiationDetails.country_code
                            )} (${originalRenegotiationDetails.country_code})`
                          : "Loading..."}
                      </Typography>
                    </Grid>
                    <Grid item xs={6}>
                      <Typography variant="subtitle2">
                        Payout Method:
                      </Typography>
                      <Typography>
                        {originalRenegotiationDetails?.payout_type
                          ? formatCase(originalRenegotiationDetails.payout_type)
                          : "Loading..."}
                      </Typography>
                    </Grid>
                    <Grid item xs={6}>
                      <Typography variant="subtitle2">Current Rate:</Typography>
                      <Typography>
                        {originalRenegotiationDetails?.current_rate
                          ? `USD ${originalRenegotiationDetails.current_rate}`
                          : "Loading..."}
                      </Typography>
                    </Grid>
                    <Grid item xs={6}>
                      <Typography variant="subtitle2">
                        Requested Rate:
                      </Typography>
                      <Typography>
                        {originalRenegotiationDetails?.requested_rate
                          ? `USD ${originalRenegotiationDetails.requested_rate}`
                          : "Loading..."}
                      </Typography>
                    </Grid>
                    <Grid item xs={6}>
                      <Typography variant="subtitle2">
                        Negotiation Level:
                      </Typography>
                      <Typography>
                        {originalRenegotiationDetails?.current_level
                          ? formatCase(
                              originalRenegotiationDetails.current_level
                            )
                          : "Loading..."}
                      </Typography>
                    </Grid>
 <Grid item xs={6}>
                      <Typography variant="subtitle2">
                        Requested By (Username):
                      </Typography>
                      <Typography>
                        {originalRenegotiationDetails?.requested_by ||
                          "Loading..."}
                      </Typography>
                    </Grid>
                    {/*
                                        PENDING CHANGE!!!!
                                        Make sure to store the comments passed by RD into the renegotiation_details table by creating a new column.
                                        Then fetch that and display it here.
                                    */}
                    <Grid item xs={12}>
                      <Typography variant="subtitle2">Comments:</Typography>
                      <Typography>
                        {renegotiationDetails?.proposal_item?.reason ||
                          "No comments provided"}
                      </Typography>
                    </Grid>
                  </Grid>
                </Box>
              )
            )}
            <form onSubmit={handleSubmit}>
              <TextField
                autoFocus
                margin="dense"
                label="Comments"
                fullWidth
                multiline
                rows={4}
                variant="outlined"
                value={comments}
                onChange={(e) => setComments(e.target.value)}
              />
            </form>
          </>
        ) : (
          <form onSubmit={handleSubmit}>
            {actionButtonText === "Approve" && hasCustomRates && (
              <Alert severity="error" sx={{ mb: 3 }}>
                Warning: Some items are at custom rate. Please validate rate by
                clicking on the view button under actions before approving!
              </Alert>
            )}
            {/* Table to show items of a proposal with warning for rates below minimum */}
            {actionButtonText === "Approve" && itemDetails.length > 0 && (
              <TableContainer component={Paper} sx={{ mb: 3 }}>
                <Table size="small">
                  <TableHead>
                    <TableRow>
                      <TableCell align="center">Country Name (Code)</TableCell>
                      <TableCell align="center">Current Rate (USD)</TableCell>
                      <TableCell align="center">Info</TableCell>
                    </TableRow>
                  </TableHead>
                  <TableBody>
                    {itemDetails.map((item) => (
                      <TableRow
                        key={item.countryCode}
                        sx={{
                          backgroundColor: item.isBelowMinimum
                            ? "rgba(255, 193, 7, 0.1)"
                            : "inherit",
                          "&:hover": {
                            backgroundColor: item.isBelowMinimum
                              ? "rgba(255, 193, 7, 0.2)"
                              : "inherit",
                          },
                        }}
                      >
                        <TableCell align="center">
                          {item.countryName} ({item.countryCode})
                        </TableCell>
                        <TableCell align="center">{item.currentRate}</TableCell>
                        <TableCell>
                          {(item.isBelowMinimum && (
                            <Box
                              sx={{
                                display: "flex",
                                alignItems: "center",
                                gap: 1,
                              }}
                            >
                              <Warning color="warning" fontSize="small" />
                              <Typography variant="body2">
                                Item rate is lower than minimum/walkaway rate of{" "}
                                <b>USD {item.minimumRate}</b>.
                              </Typography>
                            </Box>
                          )) || (
                            <Box
                              sx={{
                                display: "flex",
                                alignItems: "center",
                                gap: 1,
                              }}
                            >
                              <Alert severity="info" sx={{ mb: 0 }}>
                                Item rate is custom but greater or equal to
                                minimum/walkaway rate of{" "}
                                <b>USD {item.minimumRate}</b>.
                              </Alert>
                            </Box>
                          )}
                        </TableCell>
                      </TableRow>
                    ))}
                  </TableBody>
                </Table>
              </TableContainer>
            )}

            {/* Display the initial comments passed by the RD */}
            {proposalDetails?.proposal?.initial_comments && (
              <Box sx={{ mb: 3 }}>
                <TextField
                  label="Initial comments from Regional Director"
                  fullWidth
                  multiline
                  rows={1}
                  variant="outlined"
                  value={proposalDetails.proposal.initial_comments}
                  disabled
                />
              </Box>
            )}

            <TextField
              autoFocus
              margin="dense"
              label="Comments"
              fullWidth
              multiline
              rows={4}
              variant="outlined"
              value={comments}
              onChange={(e) => setComments(e.target.value)}
              required
              error={!comments}
              helperText={
                !comments
                  ? "Comments are required when approving non-standard rates."
                  : ""
              }
            />
          </form>
        )}
      </DialogContent>
      <DialogActions>
        <Button onClick={onClose}>Cancel</Button>
        <Button
          onClick={handleSubmit}
          variant="contained"
          color={
            actionButtonText === "Approve"
              ? "success"
              : actionButtonText === "Reject"
              ? "error"
              // : actionButtonText === "Recall"
              : actionButtonText === "Revoke"
              ? "primary"
              : "primary"
          }
          disabled={!comments.trim()}
        >
          {actionButtonText}
        </Button>
      </DialogActions>
    </Dialog>
  );
};

export default CustomModal;

import { useState, useEffect } from "react";
import {
    Box,
    Stepper,
    Step,
    StepLabel,
    Typography,
    Button,
    Alert,
    StepButton,
    Dialog,
    DialogTitle,
    DialogContent,
    DialogActions,
    Stack,
    StepConnector,
    stepConnectorClasses,
} from "@mui/material";
import { useNavigate } from "react-router-dom";
import { useSelector } from "react-redux";
import Step1 from "./Step1";
import Step2 from "./Step2";
import Step3 from "./Step3";
import SettingsIcon from "@mui/icons-material/Settings";
import GroupAddIcon from "@mui/icons-material/GroupAdd";
import VideoLabelIcon from "@mui/icons-material/VideoLabel";
import { TextBoldBig, TextNormal } from "@/common/Typography/CustomTypography";
import { Colors } from "@/theme/colors";
import { useDispatch } from "react-redux";
import { addLoader, removeLoader } from "@redux/commonSlice";
import { request } from "@utils/index";
import { addNewProposalAPI } from "@services-(pms)/proposals";
import { styled } from "@mui/styles";
import { getFeeRackRate } from "@services/feeRackRate";

const steps = [
    "Select Partner and Countries",
    // "Select Countries",
    "Configure Services",
    "Review & Submit",
];

const ColorlibConnector = styled(StepConnector)(() => ({
    [`&.${stepConnectorClasses.alternativeLabel}`]: {
        top: 22,
    },
    [`&.${stepConnectorClasses.active}`]: {
        [`& .${stepConnectorClasses.line}`]: {
            backgroundImage: `linear-gradient( 136deg, ${Colors.tpBlue} 0%, #0000DD 100%)`,
        },
    },
    [`&.${stepConnectorClasses.completed}`]: {
        [`& .${stepConnectorClasses.line}`]: {
            backgroundImage: `linear-gradient( 136deg, ${Colors.tpBlue} 0%, #0000DD 100%)`,
        },
    },
    [`& .${stepConnectorClasses.line}`]: {
        height: 3,
        border: 0,
        backgroundColor: "#eaeaf0",
        borderRadius: 1,
    },
}));

const ColorlibStepIconRoot = styled("div")(({ active, completed }) => ({
    backgroundColor: "#ccc",
    zIndex: 1,
    color: "#fff",
    width: 50,
    height: 50,
    display: "flex",
    borderRadius: "50%",
    justifyContent: "center",
    alignItems: "center",
    // ...theme.applyStyles('dark', {
    //   backgroundColor: theme.palette.grey[700],
    // }),
    ...(active || completed
        ? {
            backgroundImage: `linear-gradient( 136deg, ${Colors.tpBlue} 0%, #215881 100%)`,
            boxShadow: "0 4px 10px 0 rgba(0,0,0,.25)",
        }
        : {}),
}));

function ColorlibStepIcon(props) {
    const { active, completed, className } = props;

    const icons = {
        1: <GroupAddIcon />,
        2: <SettingsIcon />,
        3: <VideoLabelIcon />,
    };

    return (
        <ColorlibStepIconRoot
            ownerState={{ completed, active }}
            active={active}
            completed={completed}
            className={className}
        >
            {icons[String(props.icon)]}
        </ColorlibStepIconRoot>
    );
}

export default function GenerateProposal({ access }) {
    const dispatch = useDispatch();

    const countryList = useSelector((state) => state.commonReducer.countryList);

    const [activeStep, setActiveStep] = useState(0);
    const [selectedCountries, setSelectedCountries] = useState([]);
    const [error, setError] = useState("");
    const [warning, setWarning] = useState("");
    const [pricingData, setPricingData] = useState([]);
    const [completed, setCompleted] = useState({});
    const [success, setSuccess] = useState("");
    const [selectedPartner, setSelectedPartner] = useState(null);
    const [dialog, setDialog] = useState({
        open: false,
        action: () => { },
        type: "alert",
        message: "",
        title: "",
    });

    // State for preserving forecast field data
    // const [forecastFieldsData, setForecastFieldsData] = useState({});

    const [initialComments, setInitialComments] = useState("");
    const [expiryDate, setExpiryDate] = useState(() => {
        const date = new Date();
        date.setDate(date.getDate() + 30);
        return date.toISOString().split("T")[0];
    });
    const [selectedPaymentInstruments, setSelectedPaymentInstruments] =
        useState(null);
    const [countryWiseData, setCountryWiseData] = useState({});
    const [proposalName, setProposalName] = useState("");
    
    // const [hasViolations, setHasViolations] = useState(false);

    const getCountryDisplayName = (countryCode) => {
        const name = countryList.find((c) => c.country_code == countryCode);
        return name?.country_name;
    };

    const navigate = useNavigate();

    const validateVolumePrices = () => {
        const missing = [];
        if (missing.length > 0) {
            const unselectedCountries = missing
                .map(
                    (item) =>
                        `${getCountryDisplayName(item.country_code)} (${item.country_code})`
                )
                .join(", ");
            setError(`Please select volume slabs for: ${unselectedCountries}`);

            return false;
        }
        return true;
    };

    const handleNext = () => {
        let canProceed = true;
        setError(null);

        switch (activeStep) {
            case 0:
                if (!selectedPartner) {
                    setError("Please select a partner");
                    canProceed = false;
                }
                break;
            case 1:
                if (!validateVolumePrices()) {
                    canProceed = false;
                }
                if (warning) {
                    setDialog({
                        open: true,
                        action: () => {
                            const newCompleted = { ...completed };
                            newCompleted[activeStep] = true;
                            setCompleted(newCompleted);

                            setActiveStep((prevStep) => prevStep + 1);
                        },
                        type: "alert",
                        title: "Are you sure you want to proceed?",
                        message: `There are some warnings in Proposals configure.
                                  Are you sure you want to proceed?
                                  `,
                    });
                    canProceed = false;
                }
                break;
        }

        if (canProceed) {
            // Mark current step as completed
            const newCompleted = { ...completed };
            newCompleted[activeStep] = true;
            setCompleted(newCompleted);

            if (activeStep === 1) {
                // fetchPricingData();
            }

            setActiveStep((prevStep) => prevStep + 1);
        }
        return canProceed;
    };

    useEffect(() => {
        setError(null);
    }, [activeStep]);

    const handleBack = () => {
        setError("");

        // Update completed steps
        const newCompleted = { ...completed };
        delete newCompleted[activeStep - 1];
        setCompleted(newCompleted);

        setActiveStep((prevStep) => prevStep - 1);
    };

    const generateProposalName = (partner) => {
        if (!partner) return "";
        const partnerName = partner.replace(/\s/g, "_");
        const today = new Date();
        const dateStr = today.toISOString().slice(0, 10).replace(/-/g, "");
        // const randomNum = Math.floor(Math.random() * 1000000).toString().padStart(6, '0');
        // return `TPP-${partnerName}-${dateStr}-${randomNum}`;
        return `TPP-${partnerName}-${dateStr}`;
    };

    useEffect(() => {
        if (activeStep == steps.length - 1 && selectedPartner?.partner_name) {
            setProposalName(generateProposalName(selectedPartner.partner_name));
        }
    }, [activeStep, selectedPartner]);

    useEffect(() => {
        const getFeeRacks = async () => {
            if (!selectedCountries?.length) {
                return;
            }
            try {
                const queryParams = [];
                const countries = selectedCountries.join(",");
                if (countries) {
                    queryParams.push(`country=${countries}`);
                }
                const response = await request({
                    api: getFeeRackRate,
                    params: queryParams.join("&"),
                });
                if (response?.status == 200) {
                    const countryWiseRacks = {};
                    const data = response?.data?.data;
                    // setRackRate(data);

                    const payTypes =
                        selectedPaymentInstruments?.map((p) => p.value) || [];
                    data?.forEach((fee_rack) => {
                        if (
                            fee_rack.status !== "DELETED" &&
                            payTypes.includes(Number(fee_rack.payout_type))
                        ) {
                            countryWiseRacks[fee_rack.country_code] = [
                                ...(countryWiseRacks[fee_rack.country_code] || []),
                                fee_rack,
                            ];
                        }
                    });

                    setCountryWiseData(countryWiseRacks);
                }
            } catch (error) {
                // Cannot get Fee rack
            }
        };
        getFeeRacks();
    }, [selectedCountries, selectedPaymentInstruments]);

    // Update the handleSubmitProposal function to include validation for initial comments
    const handleSubmitProposal = async () => {
        try {
            if (!selectedPartner) {
                setError("Please select a partner");
                return;
            }

            // Check if any rates are below walkaway and comments are required
            // if (warning !== "" && !initialComments.trim()) {
            //     setError(
            //         "Initial comments are mandatory when rates are below minimum allowed rates"
            //     );
            //     return;
            // }

            // Find selected slab information for each item
            const proposalItems = pricingData.filter((item) => item !== null);

            const proposalData = {
                partner_id: selectedPartner.partner_id,
                proposal_name:
                    proposalName || generateProposalName(selectedPartner.partner_name),
                expiry_date: new Date(expiryDate)?.toISOString(),
                proposal_items: proposalItems,
                proposal_status: "PENDING_APPROVAL",
                ver: "1.0",
                initial_comments: initialComments,
            };

            // console.log("Submitting proposal data:", proposalData);

            try {
                dispatch(addLoader("ADD"));
                const response = await request({
                    api: addNewProposalAPI,
                    payload: proposalData,
                });
                if (response.status == 201 || response.status == 200) {
                    setSuccess("Proposal created successfully!");
                    setDialog({
                        open: true,
                        type: "msg",
                        action: () => {
                            navigate(-1);
                        },
                        title: "Proposal Created successfully!",
                    });
                }
            } catch (error) {
                //
            } finally {
                dispatch(removeLoader("ADD"));
            }

            // setTimeout(() => {
            //   navigate("/proposals");
            // }, 4000);
        } catch (error) {
            console.error("Proposal creation error:", error);
            setError(
                "Failed to create proposal: " +
                (error.response?.data?.error || error.message)
            );
        }
    };

    const renderStepContent = (step) => {
        switch (step) {
            case 0:
                return (
                    <Step1
                        selectedCountries={selectedCountries}
                        setSelectedCountries={(data) => {
                            // setPricingData([]);
                            setSelectedCountries(() => data);
                        }}
                        selectedPartner={selectedPartner}
                        setSelectedPartner={setSelectedPartner}
                        selectedPaymentInstruments={selectedPaymentInstruments}
                        setSelectedPaymentInstruments={setSelectedPaymentInstruments}
                    />
                );
            case 1:
                return (
                    <Step2
                        selectedCountries={selectedCountries}
                        selectedPartner={selectedPartner}
                        pricingData={pricingData}
                        setPricingData={setPricingData}
                        setError={setWarning}
                        countryWiseData={countryWiseData}
                        selectedPaymentInstruments={selectedPaymentInstruments}
                        handleNext={handleNext}
                    />
                );
            case 2:
                return (
                    <Step3
                        error={error}
                        expiryDate={expiryDate}
                        handleSubmitProposal={handleSubmitProposal}
                        initialComments={initialComments}
                        proposalName={proposalName}
                        selectedPartner={selectedPartner}
                        setExpiryDate={setExpiryDate}
                        setInitialComments={setInitialComments}
                        success={success}
                        pricingData={pricingData}
                        hasViolations={warning !== ""}
                    />
                );
            default:
                return null;
        }
    };

    return (
        <Box sx={{ p: 3 }}>
            <TextBoldBig fontSize={26} gutterBottom>
                Generate Proposal
            </TextBoldBig>

            <Stepper
                alternativeLabel
                connector={<ColorlibConnector />}
                activeStep={activeStep}
                sx={{ mb: 4, mt: 4 }}
            >
                {steps.map((label, index) => (
                    <Step key={label} completed={completed[index]}>
                        <StepButton
                            onClick={() => {
                                let canProceed = true;

                                // Check conditions based on target step
                                if (index === 1) {
                                    // Configure Services
                                    // Allow if either:
                                    // 1. Step was previously completed OR
                                    // 2. Countries are selected in step 1
                                    if (!completed[index] && selectedCountries.length === 0) {
                                        setError("Please select at least one country");
                                        canProceed = false;
                                    }
                                }

                                // Don't allow clicking future uncompleted steps
                                if (index > activeStep && !completed[index]) {
                                    canProceed = false;
                                }

                                if (canProceed) {
                                    setError("");
                                    
                                    // If moving to step 2 and it wasn't completed, fetch pricing data
                                    if (index === 2 && !completed[2]) {
                                        // fetchPricingData();
                                    }

                                    setActiveStep(index);
                                }
                            }}
                            disabled={index > activeStep && !completed[index]}
                            sx={{
                                cursor:
                                    index <= activeStep || completed[index]
                                        ? "pointer"
                                        : "not-allowed",
                            }}
                        >
                            <StepLabel slots={{ stepIcon: ColorlibStepIcon }}>
                                {label}
                            </StepLabel>
                        </StepButton>
                    </Step>
                ))}
            </Stepper>
            {activeStep > 0 && activeStep <= steps.length && (
                <Box onClick={handleBack} sx={{ mr: 1, cursor: "pointer", width: 'fit-content' }}>
                    {"<   Back"}
                </Box>
            )}

            {error && <Alert severity="error">{error}</Alert>}
            {warning && <Alert severity="warning">{warning}</Alert>}
            {renderStepContent(activeStep)}

            <Box sx={{ display: "flex", justifyContent: "flex-end", mt: 2 }}>
                {/* {activeStep > 0 && activeStep != steps.length - 1 && (
                    <Button onClick={handleBack} sx={{ mr: 1 }}>
                        Back
                    </Button>
                )} */}
                {activeStep < steps.length - 1 && (
                    <Button
                        variant="contained"
                        onClick={handleNext}
                        disabled={
                            (activeStep === 0 &&
                                (!selectedPartner?.partner_id ||
                                    !selectedCountries.length ||
                                    !selectedPaymentInstruments?.length))
                            // || (activeStep === 1 && (
                            //     pricingData.length === 0 ||
                            //     pricingData.some(item => {
                            //         if (item.calculation_mode === "percentage") {
                            //             return !item.fee_percentage || parseFloat(item.fee_percentage) === 0;
                            //         } else {
                            //             return !item.fee || parseFloat(item.fee) === 0;
                            //         }
                            //     })
                            // ))
                        }
                    >
                        Next
                    </Button>
                )}
            </Box>
            {dialog.open && (
                <>
                    {dialog.type == "alert" && (
                        <Dialog
                            open={dialog.open}
                            onClose={() => {
                                setDialog({
                                    open: false,
                                    message: "",
                                    title: "",
                                    type: "",
                                });
                            }}
                        >
                            <DialogTitle sx={{ bgcolor: Colors.tpBlue, color: "white" }}>
                                {dialog.title}
                            </DialogTitle>
                            <DialogContent sx={{ mt: 1 }}>
                                <TextNormal>{dialog.message}</TextNormal>
                            </DialogContent>
                            <DialogActions>
                                <Stack direction="row" justifyContent="end" gap={2}>
                                    <Button
                                        variant="outlined"
                                        onClick={() => {
                                            setDialog({
                                                open: false,
                                                message: "",
                                                title: "",
                                                type: "",
                                            });
                                        }}
                                    >
                                        Cancel
                                    </Button>
                                    <Button
                                        variant="contained"
                                        onClick={() => {
                                            dialog.action?.();
                                            setDialog({
                                                open: false,
                                                message: "",
                                                title: "",
                                                type: "",
                                            });
                                        }}
                                    >
                                        OK
                                    </Button>
                                </Stack>
                            </DialogActions>
                        </Dialog>
                    )}
                    {dialog.type == "msg" && (
                        <Dialog
                            open={dialog.open}
                            onClose={() => {
                                dialog.action?.();
                            }}
                            sx={{
                                minWidth: "300px",
                                minHeight: "300px",
                            }}
                        >
                            <DialogContent
                                sx={{
                                    bgcolor: Colors.tpBlue,
                                    color: "white",
                                }}
                            >
                                <Stack gap={2} alignItems={"center"} justifyContent={"center"}>
                                    <Typography variant="h5" color="white">
                                        {dialog.title || dialog.message}
                                    </Typography>
                                    <Button
                                        variant="text"
                                        sx={{ bgcolor: "white", maxWidth: "60px" }}
                                        fullWidth={false}
                                        onClick={dialog.action}
                                    >
                                        OK
                                    </Button>
                                </Stack>
                            </DialogContent>
                        </Dialog>
                    )}
                </>
            )}
        </Box>
    );
}

import { Box, Grid2, Stack, Typography } from "@mui/material";
import { useSelector } from "react-redux";
import { useState, useEffect, useMemo } from "react";
import { useDispatch } from "react-redux";
import { addLoader, removeLoader } from "@redux/commonSlice";
import { request } from "@utils/index";
import { getFeeRackSpeedTexts } from "@services/feeRackRate";
import { getAvailableCountriesAPI } from "@services-(pms)/proposals";
import Dropdown from "@/common/Dropdown/Dropdown";
import MultiSelectDropdown from "@/common/MultiselectDropdown.js/MultiSelect";
import { hardcodedCountries } from "@modules-(onboarding)/configuration/FeeRackRate/FeeRackRate/defaults";

function Step1({
    selectedPartner,
    setSelectedPartner,
    selectedCountries,
    setSelectedCountries,
    selectedPaymentInstruments,
    setSelectedPaymentInstruments,
}) {
    const dispatch = useDispatch();

    const countryList = useSelector((state) => state.commonReducer.countryList);
    const partnerList = useSelector((state) => state.commonReducer.partnerList);

    const [countries, setCountries] = useState(null);
    const [allPartners, setAllPartners] = useState([]);
    const [paymentInstruments, setPaymentInstruments] = useState([]);

    const countriesOptions = useMemo(() => countryList?.map((country) => ({
        label: country.country_name,
        value: country.country_code,
    })), [countryList]);

    useEffect(() => {
        const getSpeedTextAPICall = async () => {
            dispatch(addLoader("SPEED_TEXT"));
            try {
                const response = await request({
                    api: getFeeRackSpeedTexts,
                });
                if (response.status === 200) {
                    const paymentInstruments = response.data.data2?.map((item) => ({
                        label: item.type,
                        value: item.payment_instrument_id,
                        name: item.payment_instrument_name,
                        type: item.type?.toUpperCase(),
                    }));
                    setPaymentInstruments(paymentInstruments);
                }
            } finally {
                dispatch(removeLoader("SPEED_TEXT"));
            }
        };
        getSpeedTextAPICall();
    }, [dispatch]);

    useEffect(() => {
        const partners = partnerList?.map((d) => ({
            ...d,
            label: d.partner_name,
            value: d.partner_id,
        }));
        setAllPartners([...(partners || [])]);
    }, [partnerList]);

    useEffect(() => {
        const fetchCountries = async () => {
            try {
                dispatch(addLoader("FETCH_COUNTRIES"));

                const response = await request({
                    api: getAvailableCountriesAPI,
                    params: { status: "ACTIVE" },
                });

                console.log("Countries API response:", response);

                if (response.status === 200) {
                    const countryCodes = response.data.data || [];

                    // Format countries as objects with label and value
                    const formattedCountries = countryCodes.map(countryCode => {
                        const country = [...hardcodedCountries, ...countriesOptions].find(c => c.value === countryCode);
                        return {
                          label: `${country.label} (${countryCode})`,
                          value: countryCode,
                        };
                    });

                    setCountries(formattedCountries || []);
                }
            } catch (error) {
                console.error("Error fetching countries:", error);
            } finally {
                dispatch(removeLoader("FETCH_COUNTRIES"));
            }
        };
    
        fetchCountries();
    }, [dispatch, countriesOptions]);

    // useEffect(() => {
    //     setCountries(
    //         countryList?.map((item) => ({
    //             label: `${item.country_name} (${item.country_code})`,
    //             value: item.country_code,
    //         }))
    //     );
    // }, [countryList]);

    const getCountryDisplayName = (countryCode) => {
        // Use utility function for consistent SEPA handling
        return [...hardcodedCountries, ...countriesOptions].find(c => c.value === countryCode)?.label;
    };

    return (
        <Stack sx={{ mt: 2 }} gap={2}>
            <Grid2 container>
                <Grid2 item size={{ md: 6, lg: 4, xs: 12 }}>
                    <Box mx={2}>
                        <Typography variant="h6" gutterBottom>
                            Select Partner
                        </Typography>
                        <Dropdown
                            multiple={false}
                            options={allPartners}
                            selectedOption={selectedPartner}
                            checkBox={false}
                            zIndex={1100}
                            height={"48px"}
                            size={"medium"}
                            label="Select Partner"
                            onChange={(e, value) => {
                                setSelectedPartner(value);
                            }}
                        />
                    </Box>
                </Grid2>
                <Grid2 item size={{ xs: 12, md: 6, lg: 4 }}>
                    <Box mx={2}>
                        <Typography variant="h6" gutterBottom>
                            Select Countries
                        </Typography>
                        <MultiSelectDropdown
                            showSelectAllOption
                            maxChip={3}
                            chipWidth="120px"
                            options={countries}
                            selectedOption={
                                selectedCountries?.map((c) => ({
                                    label: `${getCountryDisplayName(c)} (${c})`,
                                    value: c,
                                })) || null
                            }
                            multiple
                            checkBox={false}
                            zIndex={1100}
                            height={"48px"}
                            label="Select Countries"
                            onChange={(e, value) => {
                                setSelectedCountries(value?.map((v) => v.value));
                            }}
                        />
                    </Box>
                </Grid2>
                <Grid2 item size={{ xs: 12, md: 6, lg: 4 }}>
                    <Box mx={2}>
                        <Typography variant="h6" gutterBottom>
                            Select Payout Types
                        </Typography>
                        <MultiSelectDropdown
                            options={paymentInstruments}
                            selectedOption={selectedPaymentInstruments}
                            maxChip={3}
                            multiple
                            checkBox={false}
                            zIndex={1100}
                            height={"48px"}
                            label="Select Payout types"
                            onChange={(e, values) => {
                                if (values.find((v) => v.type === "ALL")) {
                                    return setSelectedPaymentInstruments([...paymentInstruments]);
                                }
                                setSelectedPaymentInstruments(values);
                            }}
                        />
                    </Box>
                </Grid2>
            </Grid2>
        </Stack>
    );
}

export default Step1;

import { Colors } from "@/theme/colors";
import { Box, Button, Typography } from "@mui/material";
import ItemsTables from "../ItemsTables";
import { useCallback, useEffect, useMemo, useState } from "react";

import uniqWith from "lodash/uniqWith";
import Dropdown from "@/common/Dropdown/Dropdown";
import CommonButton from "@/common/Button/CommonButton";
import { feeValueOptions } from "../defaults";

function Step2({
  selectedPartner,
  pricingData,
  setPricingData,
  setError,
  countryWiseData,
  handleNext,
  // selectedPaymentInstruments,
}) {
  // const [loadingPricing, setLoading] = useState(true);
  // const [countryWiseData, setCountryWiseData] = useState({});

  const [feeValue, setFeeValue] = useState(feeValueOptions[0]);

  const someItemsSelected = useMemo(() => {
    return pricingData.some((item) => item?.check_box === true);
  }, [pricingData]);

    const handleFeeValueApplyToAll = useCallback((feeValue, index, check, type) => {
      // Make a copy to avoid mutating the original
      const updatedData = pricingData.map((item, idx) => {
        // Skip processing if this item has been manually edited by the user
        if (check && index !== undefined && index !== idx) {
          console.log(`Skipping auto-processing for user-edited item: ${item.country_code}`);
          return item;
        }
        if (check == undefined && !item.check_box) {
          console.log(`Skipping auto-processing for user-edited item: ${item.country_code}`);
          return item;
        }
        item.userEdited = true;

        // Handle mixed rates with custom_fee_enable
        if (item.custom_fee_enable === 1 || item.fee_type === "MIXED") {
          console.log(`Processing mixed rate for ${item.country_code}, calculation_mode: ${item.calculation_mode}`);
          const feeData = item[feeValue.value];
          const fee2Data = item[feeValue.value2];

          if (item.calculation_mode === "percentage") {
            // For percentage primary mode, secondary is flat
            const percentageValue = feeData || 0;
            const flatValue = (() => {
              try {
                // Try to parse the custom_proposed_fee as JSON
                const customFee = fee2Data ? JSON.parse(fee2Data) : null;
                // If parsing succeeded and has a fees property, use it
                if (customFee && customFee.fees) {
                  return parseFloat(customFee.fees);
                }
              } catch (e) {
                // If JSON parsing fails, fall back to direct field value
                console.log("Failed to parse custom fee JSON", e);
              }
              return parseFloat(item.fee_flat || 0);
            })();

            const updatedItem = {
              ...item,
              fee_type: "MIXED",
                        fee_percentage: index != undefined ? type === 'percentage' ? percentageValue : item.fee_percentage : percentageValue,
                        fee: index != undefined ? type === 'flat' ? flatValue : item.fee : percentageValue, // Set fee equal to fee_percentage for percentage mode
                        fee_flat: index != undefined ? type === 'flat' ? flatValue : item.fee_flat : flatValue, // Explicitly set fee_flat
            };
            console.log(`Updated percentage-primary mixed rate: flat=${updatedItem.fee_flat}, percentage=${updatedItem.fee_percentage}`);
            return updatedItem;
          } else {
            // For flat primary mode, secondary is percentage
            const flatValue = feeData || 0;
            const percentageValue = (() => {
              try {
                // Try to parse the custom_proposed_fee as JSON
                const customFee = fee2Data ? JSON.parse(fee2Data) : null;
                // If parsing succeeded and has a fees property, use it
                if (customFee && customFee.fees) {
                  return parseFloat(customFee.fees);
                }
              } catch (e) {
                // If JSON parsing fails, fall back to direct field value
                console.log("Failed to parse custom fee JSON", e);
              }
              return parseFloat(item.fee_percentage || 0);
            })();

            const updatedItem = {
              ...item,
              fee_type: "MIXED",
                        fee: index != undefined ? type === 'flat' ? flatValue : item.fee : flatValue,
                        fee_flat: index != undefined ? type === 'flat' ? flatValue : item.fee_flat : flatValue, // Explicitly set fee_flat
                        fee_percentage: index != undefined ? type === 'percentage' ? percentageValue : item.fee_percentage : percentageValue, // Explicitly set fee_percentage
            };
            console.log(`Updated flat-primary mixed rate: flat=${updatedItem.fee_flat}, percentage=${updatedItem.fee_percentage}`);
            return updatedItem;
          }
        }
        // Handle standard percentage-only rates
        else if (item.calculation_mode === "percentage") {
          return {
            ...item,
            fee_type: "PERCENTAGE",
                    fee_percentage: index != undefined ? type === 'percentage' ? item[feeValue.value] : item.fee_percentage : (item[feeValue.value] || 0),
                    fee: index != undefined ? type === 'flat' ? item.fee_flat : item.fee : (item[feeValue.value] || 0), // Set fee equal to fee_percentage for percentage mode
          };
        }
        // Handle standard flat-only rates
        else {
          return {
            ...item,
            fee_type: "FLAT",
                    fee: index != undefined ? type === 'flat' ? item[feeValue.value] : item.fee : (item[feeValue.value] || 0),
            fee_percentage: null,
          };
        }
      });

      // Only update if there's a difference to avoid infinite loops
      if (JSON.stringify(updatedData) !== JSON.stringify(pricingData)) {
        console.log("Updating pricing data with processed mixed rates");
        setPricingData(updatedData);
      }
    }, [pricingData, setPricingData]);

  useEffect(() => {
    const data = [];
    for (const key in countryWiseData) {
      const d = countryWiseData[key];
            const uniques = uniqWith(
                d,
                (a, b) =>
                    a.payout_type == b.payout_type &&
                    a.payout_currency == b.payout_currency &&
                    a.coverage == b.coverage &&
                    a.transaction_type == b.transaction_type
            );
      data.push(...uniques);
    }

    // Ensure each row has a unique id for proper pagination
    const newDataWithIds = data.map((item, index) => ({
      ...item,
      id: item.id || item.frr_id || `${item.country_code}_${item.payout_type}_${index}`
    }));

    setPricingData((prevPricingData) => {
      // If no previous data, just set the new data
      if (!prevPricingData || prevPricingData.length === 0) {
        return newDataWithIds;
      }

      // Merge logic:
      // Loop through the NEW data (which reflects the currently selected countries/filters).
      // For each new item, check if we already have it in the OLD data (preserved state).
      // If yes, keep the OLD item (with user edits).
      // If no, use the NEW item.
      const mergedData = newDataWithIds.map(newItem => {
        const existingItem = prevPricingData.find(oldItem =>
          oldItem.country_code === newItem.country_code &&
          oldItem.payout_type === newItem.payout_type &&
          oldItem.payout_currency === newItem.payout_currency
        );

        if (existingItem) {
          // Keep the existing item to preserve user edits (fees, volume, etc.)
          // We might want to update some fields from backend if they are NOT user editable,
          // but usually keeping the state object is safer for persistence.
          return existingItem;
        }
        return newItem;
      });

      // Check if data actually changed to verify we aren't losing anything
      // or if we need to remove items (e.g. user REMOVED a country in Step 1)
      // The map above only includes items from newDataWithIds, which is derived from countryWiseData.
      // So if a country was removed from countryWiseData, it naturally won't be in mergedData used here.
      // This correctly handles both Addition and Removal of countries.

      // Optimization: Only update state if JSON stringified result is different
      if (JSON.stringify(mergedData) !== JSON.stringify(prevPricingData)) {
        return mergedData;
      }

      return prevPricingData;
    });
  }, [countryWiseData, setPricingData]);

  useEffect(() => {
    if (pricingData?.length > 0) {
      // Make a copy to avoid mutating the original
            const updatedData = pricingData.map(item => {
        // Skip processing if this item has been manually edited by the user
        if (item.userEdited) {
          console.log(`Skipping auto-processing for user-edited item: ${item.country_code}`);
          return item;
        }

        // Handle mixed rates with custom_fee_enable
        if (item.custom_fee_enable === 1 || item.fee_type === "MIXED") {
          console.log(`Processing mixed rate for ${item.country_code}, calculation_mode: ${item.calculation_mode}`);

          if (item.calculation_mode === "percentage") {
            // For percentage primary mode, secondary is flat
            const percentageValue = item.fee_percentage || item.fee || item.proposed_fee || 0;
            const flatValue = (() => {
              try {
                // Try to parse the custom_proposed_fee as JSON
                const customFee = item.custom_proposed_fee ? JSON.parse(item.custom_proposed_fee) : null;
                // If parsing succeeded and has a fees property, use it
                if (customFee && customFee.fees) {
                  return parseFloat(customFee.fees);
                }
              } catch (e) {
                // If JSON parsing fails, fall back to direct field value
                console.log("Failed to parse custom fee JSON", e);
              }
              return parseFloat(item.fee_flat || 0);
            })();

            const updatedItem = {
              ...item,
              fee_type: "MIXED",
              fee_percentage: percentageValue,
              fee: percentageValue, // Set fee equal to fee_percentage for percentage mode
              fee_flat: flatValue, // Explicitly set fee_flat
            };
            console.log(`Updated percentage-primary mixed rate: flat=${updatedItem.fee_flat}, percentage=${updatedItem.fee_percentage}`);
            return updatedItem;
          } else {
            // For flat primary mode, secondary is percentage
            const flatValue = item.fee || item.proposed_fee || 0;
            const percentageValue = (() => {
              try {
                // Try to parse the custom_proposed_fee as JSON
                const customFee = item.custom_proposed_fee ? JSON.parse(item.custom_proposed_fee) : null;
                // If parsing succeeded and has a fees property, use it
                if (customFee && customFee.fees) {
                  return parseFloat(customFee.fees);
                }
              } catch (e) {
                // If JSON parsing fails, fall back to direct field value
                console.log("Failed to parse custom fee JSON", e);
              }
              return parseFloat(item.fee_percentage || 0);
            })();

            const updatedItem = {
              ...item,
              fee_type: "MIXED",
              fee: flatValue,
              fee_flat: flatValue, // Explicitly set fee_flat
              fee_percentage: percentageValue, // Explicitly set fee_percentage
            };
            console.log(`Updated flat-primary mixed rate: flat=${updatedItem.fee_flat}, percentage=${updatedItem.fee_percentage}`);
            return updatedItem;
          }
        }
        // Handle standard percentage-only rates
        else if (item.calculation_mode === "percentage") {
          return {
            ...item,
            fee_type: "PERCENTAGE",
            fee_percentage: item.fee_percentage || item.fee || item.proposed_fee || 0,
            fee: item.fee_percentage || item.fee || item.proposed_fee || 0, // Set fee equal to fee_percentage for percentage mode
          };
        }
        // Handle standard flat-only rates
        else {
          return {
            ...item,
            fee_type: "FLAT",
            fee: item.fee || item.proposed_fee || 0,
                        fee_percentage: null
          };
        }
      });

      // Only update if there's a difference to avoid infinite loops
      if (JSON.stringify(updatedData) !== JSON.stringify(pricingData)) {
        console.log("Updating pricing data with processed mixed rates");
        setPricingData(updatedData);
      }
    }
  }, [pricingData, setPricingData]);

  // This effect checks for below-minimum rates and shows a warning
  useEffect(() => {
    if (pricingData?.length > 0) {
      // Find rows with rates below minimum (including mixed rates)
            const belowMinimumRows = pricingData.filter(item => {
        // Check mixed rates
        if (item.custom_fee_enable === 1 || item.fee_type === "MIXED") {
          // Parse minimum rates - handle both new and legacy JSON structures
          let minimumFlat = 0;
          let minimumPercentage = 0;

          if (item.custom_minimum_fee) {
            try {
              const customMinimum = JSON.parse(item.custom_minimum_fee);

              // Check for new structure: {"feeName":"CUSTOM_FEE","calculationMode":"percentage","fees":"7"}
              if (customMinimum.calculationMode && customMinimum.fees) {
                const minValue = parseFloat(customMinimum.fees || 0);
                                if (customMinimum.calculationMode === 'percentage') {
                  // The JSON contains the percentage minimum
                  minimumPercentage = minValue;
                  // Get flat minimum from regular minimum_fee field
                  minimumFlat = parseFloat(item.minimum_fee || 0);
                                } else if (customMinimum.calculationMode === 'flat') {
                  // The JSON contains the flat minimum
                  minimumFlat = minValue;
                  // Try to get percentage minimum from other sources
                  minimumPercentage = parseFloat(item.minimum_fee_percentage || 0);
                }
              }
              // Check for legacy structure: {"flat_fee":"15","percentage_fee":"7"}
              else if (customMinimum.flat_fee || customMinimum.percentage_fee) {
                minimumFlat = parseFloat(customMinimum.flat_fee || 0);
                minimumPercentage = parseFloat(customMinimum.percentage_fee || 0);
                            }
                            else {
                // Fallback to individual minimum fields
                minimumFlat = parseFloat(item.minimum_fee || 0);
                minimumPercentage = parseFloat(item.minimum_fee_percentage || 0);
              }
            } catch (e) {
              console.log("Failed to parse custom_minimum_fee JSON:", e);
              // Fallback to individual minimum fields if JSON parsing fails
              minimumFlat = parseFloat(item.minimum_fee || 0);
              minimumPercentage = parseFloat(item.minimum_fee_percentage || 0);
            }
          } else {
            // Fallback to individual minimum fields
            minimumFlat = parseFloat(item.minimum_fee || 0);
            minimumPercentage = parseFloat(item.minimum_fee_percentage || 0);
          }

          console.log(`Mixed rate warning check for ${item.country_code}: minimumFlat=${minimumFlat}, minimumPercentage=${minimumPercentage}`);

          // Get current proposed rates for both components
          const proposedFlat = parseFloat(item.fee_flat || 0);
          const proposedPercentage = parseFloat(item.fee_percentage || 0);

          console.log(`Mixed rate proposed values: proposedFlat=${proposedFlat}, proposedPercentage=${proposedPercentage}`);

          // Both components must be >= minimum for mixed rates
          const flatAcceptable = proposedFlat >= minimumFlat;
          const percentageAcceptable = proposedPercentage >= minimumPercentage;

          const isBelowMinimum = !(flatAcceptable && percentageAcceptable);
                    console.log(`Mixed rate validation: flatAcceptable=${flatAcceptable}, percentageAcceptable=${percentageAcceptable}, isBelowMinimum=${isBelowMinimum}`);

          return isBelowMinimum;
        }

        // Check flat rates
                if (item.calculation_mode === "flat" ||
                    (item.fee_type === "FLAT" && item.calculation_mode !== "percentage")) {
          return parseFloat(item.fee || 0) < parseFloat(item.minimum_fee_flat || 0);
        }

        // Check percentage rates
                if (item.calculation_mode === "percentage" ||
                    (item.fee_type === "PERCENTAGE" && item.calculation_mode !== "flat")) {
          return parseFloat(item.fee_percentage || 0) < parseFloat(item.minimum_fee_percentage || 0);
        }

        return false;
      });

      // If there are any below-minimum rates, show a warning
      if (belowMinimumRows.length > 0) {
        // Get country names for affected rows
                const countries = belowMinimumRows.map(row => {
          return row.country_code;
        });

        // Set warning message
        setError(`Warning: Rates are below minimum for the following countries: ${countries.join(", ")}. These rows are highlighted in red.`);
      } else {
        // Clear warning if no below-minimum rates
        setError("");
      }
    }
  }, [pricingData, setError]);

  const getVelocityTypes = ({ country_code, payout_currency, payout_type, coverage }) => {
    const data = countryWiseData[country_code]?.filter(
            (frr) =>
                frr.payout_currency == payout_currency && frr.payout_type == payout_type && frr.coverage == coverage
    );
    if (!data) return [];
    const values = [];
    data.forEach((c) => {
      if (!values.find((v) => v == (c.velocity_type || 2))) {
        values.push(c.velocity_type || 2);
      }
    });
    return values;
  };

    const getVelocityPeriods = ({
        country_code,
        payout_currency,
        payout_type,
        coverage,
    }) => {
    const data = countryWiseData[country_code]?.filter(
            (frr) =>
                frr.payout_currency == payout_currency && frr.payout_type == payout_type && frr.coverage == coverage
    );
    if (!data) return [];
    const values = [];

    data.forEach((c) => {
            if (c.velocity_period === 0 &&
                !values
                    .find((v) => v == (c.velocity_period))
            ) {
        values.push(c.velocity_period);
            }
            else if (!values.find((v) => v == (c.velocity_period || 2))) {
        values.push(c.velocity_period || 2);
      }
    });
    return values;
  };

  const getRanges = (row) => {
    const { country_code, payout_currency, payout_type, fee_type, coverage } = row;
    const data = countryWiseData?.[country_code]?.filter(
            (frr) =>
                frr.payout_currency == payout_currency && frr.payout_type == payout_type && frr.coverage == coverage
    );
    if (!data) return [];
    const t_options = getVelocityTypes(row);
    const p_options = getVelocityPeriods(row);
    const ranges = [];
    const default_t = t_options?.[0]?.value || 2;
    const default_p = p_options?.[0]?.value || 2;

        const isMixed = fee_type == 'MIXED' || row.custom_fee_enable == 1;

    data.forEach((d) => {
      if (
                (d.velocity_period || default_p) ==
                (row.velocity_period || default_p) &&
        (d.velocity_type || default_t) == (row.velocity_type || default_t) &&
        (!row.payout_type || d.payout_type == row.payout_type) &&
                (isMixed ? d.custom_fee_enable : (d.calculation_mode?.toUpperCase() == fee_type && d.custom_fee_enable != 1))
      )
        ranges.push({
          min_value: d.min_value || 0,
          max_value: d.max_value || 999999999,
          id: d.frr_id,
          otherData: d,
        });
    });

    console.log("Range", data, ranges);
    return ranges;
  };

  const getCalcModes = ({ country_code, payout_currency, payout_type, coverage }) => {
    const data = countryWiseData?.[country_code]?.filter(
            (frr) =>
                frr.payout_currency == payout_currency && frr.payout_type == payout_type && frr.coverage == coverage
    );
    if (!data) return [];
    const values = [];
    data.forEach((d) => {
      if (!values.find((v) => v.value == d.calculation_mode)) {
        values.push({
          label: d.calculation_mode,
          value: d.calculation_mode,
        });
      }
    });
    return values;
  };

  // Function to automatically update slab range and fee based on committed volume
  const updateSlabBasedOnCommittedVolume = (index, commitedVolume) => {
    const volume = parseInt(commitedVolume || 0, 10);

    setPricingData((prev) => {
      const updatedData = [...prev];
      const currentRow = updatedData[index];

      if (!currentRow) return prev;

      // Get available ranges for this row
      const availableRanges = getRanges(currentRow);

      if (!availableRanges || availableRanges.length === 0) {
                console.log('No available ranges found for row:', currentRow);
        return prev;
      }

      // Find the appropriate slab range that contains the committed volume
            const appropriateRange = availableRanges.find(range => {
        const minValue = range.min_value || 0;
        const maxValue = range.max_value || 999999999;
        return volume >= minValue && volume <= maxValue;
      });

      if (appropriateRange) {
        console.log(`Updating row ${index} to range:`, appropriateRange);
        console.log(`Volume ${volume} fits in range ${appropriateRange.min_value} - ${appropriateRange.max_value}`);

        // Update the row with the new slab range data
        const updatedRow = {
          ...currentRow,
          fee_rack_rate_id: appropriateRange.id,
          min_value: appropriateRange.min_value,
          max_value: appropriateRange.max_value,
          // Update fee data from the new slab
          ...appropriateRange.otherData,
        };

        // Handle mixed rates properly
        if (appropriateRange.otherData.custom_fee_enable === 1) {
          updatedRow.fee_type = "MIXED";
          updatedRow.custom_fee_enable = 1;

          if (appropriateRange.otherData.calculation_mode === "percentage") {
            // For percentage primary mode
            updatedRow.fee_percentage = appropriateRange.otherData.proposed_fee || appropriateRange.otherData.fee_percentage || 0;
            updatedRow.fee = updatedRow.fee_percentage;

            // Parse flat component from JSON
            try {
                            const customFee = appropriateRange.otherData.custom_proposed_fee ?
                                JSON.parse(appropriateRange.otherData.custom_proposed_fee) : null;
              updatedRow.fee_flat = customFee?.fees || 0;
            } catch (e) {
              console.log("Failed to parse custom fee JSON", e);
              updatedRow.fee_flat = 0;
            }
          } else {
            // For flat primary mode
            updatedRow.fee = appropriateRange.otherData.proposed_fee || appropriateRange.otherData.fee || 0;

            // Parse percentage component from JSON
            try {
                            const customFee = appropriateRange.otherData.custom_proposed_fee ?
                                JSON.parse(appropriateRange.otherData.custom_proposed_fee) : null;
              updatedRow.fee_percentage = customFee?.fees || 0;
            } catch (e) {
              console.log("Failed to parse custom fee JSON", e);
              updatedRow.fee_percentage = 0;
            }
          }
        } else {
          // Handle standard (non-mixed) rates
          if (appropriateRange.otherData.calculation_mode === "percentage") {
            updatedRow.fee_percentage = appropriateRange.otherData.proposed_fee || appropriateRange.otherData.fee_percentage || 0;
            updatedRow.fee = updatedRow.fee_percentage;
          } else {
            updatedRow.fee = appropriateRange.otherData.proposed_fee || appropriateRange.otherData.fee || 0;
            updatedRow.fee_percentage = null;
          }
        }

        updatedData[index] = updatedRow;
        console.log("Updated row data:", updatedRow);
      } else {
        console.log(`No appropriate range found for volume ${volume} in available ranges:`, availableRanges);
      }

      return updatedData;
    });
  };

  const getCoverageOptions = ({ country_code, payout_type, payout_currency }) => {
    const coverageOptions = countryWiseData[country_code]?.filter((item) => item.payout_type === payout_type && item.payout_currency === payout_currency);
    console.log("DEBUG - countryWiseData", countryWiseData);
    const values = [];

    coverageOptions?.forEach((item) => {
            if (!values.some(value => value.coverage === item.coverage)) {
        values.push({ coverage: item.coverage, frr_id: item.id || item.frr_id });
      }
    });

    console.log("DEBUG - availableCoverageValues", values);

    return values;
    }

  const getTransactionTypeOptions = ({ country_code, payout_type, payout_currency, coverage }) => {
    const transactionTypeOptions = countryWiseData[country_code]?.filter(
      (item) => item.payout_type === payout_type && item.payout_currency === payout_currency && item.coverage === coverage
    );
    const values = [];

    transactionTypeOptions?.forEach((item) => {
            if (!values.some(value => value.transaction_type === item.transaction_type)) {
        values.push({ transaction_type: item.transaction_type, frr_id: item.id || item.frr_id });
      }
    });

    return values;
    }

  return (
    <Box sx={{ mt: 4 }}>
      <Box
        sx={{
          display: "flex",
          justifyContent: "space-between",
          alignItems: "center",
          mb: 2,
        }}
      >
        <Box
          sx={{
            display: "flex",
            alignItems: "center",
            gap: 2,
          }}
        >
          <Typography variant="h6" sx={{ color: Colors.tpBlue }}>
            Configure Services
          </Typography>
          {selectedPartner && (
            <Typography variant="h6" sx={{ color: Colors.tpBlue }}>
              | Partner: {selectedPartner.partner_name}
            </Typography>
          )}
        </Box>
        <Box sx={{ display: "flex", alignItems: "right", gap: 2 }}>
                    {someItemsSelected && <Box
              sx={{
                display: "flex",
                alignItems: "center",
                gap: 2,
              }}
            >
              <Dropdown
                name={"fee_value"}
                width="200px"
                options={feeValueOptions}
                selectedOption={feeValue || null}
                onChange={(e, v) => {
                  setFeeValue(v);
                }}
              />
                        <CommonButton
                            variant="contained"
                            color="primary"
                            disabled={!someItemsSelected}
                            onClick={() => handleFeeValueApplyToAll(feeValue)}
                        >
                Apply
              </CommonButton>
                    </Box>}
                    {handleNext && typeof handleNext === "function" &&
                        <Button
                            variant="contained"
                            onClick={handleNext}
                        >
              Next
                        </Button>}
        </Box>
      </Box>

      <ItemsTables
        showActions
        data={pricingData}
        setData={setPricingData}
        getVelocityPeriods={getVelocityPeriods}
        getVelocityTypes={getVelocityTypes}
        getRanges={getRanges}
        getCalcModes={getCalcModes}
        getCoverageOptions={getCoverageOptions}
        getTransactionTypeOptions={getTransactionTypeOptions}
        showForecastFields={true}
        onEditVolume={false}
        enablePagination={true}
        itemsPerPage={25}
        updateSlabBasedOnCommittedVolume={updateSlabBasedOnCommittedVolume}
        handleFeeValueApplyToAll={handleFeeValueApplyToAll}
      />
    </Box>
  );
}

export default Step2;

import TextFieldComponent from "@/common/TextField/TextFieldComponent";
import {
    Box,
    Button,
    Card,
    CardContent,
    Grid,
    Typography,
} from "@mui/material";
import { Alert } from "@mui/lab";
import ItemsTables from "../ItemsTables";
import BasicDatePicker from "@/common/DateAndTime/DatePicker";

function Step3({
    selectedPartner,
    error,
    success,
    initialComments,
    setInitialComments,
    expiryDate,
    setExpiryDate,
    proposalName,
    handleSubmitProposal,
    pricingData,
    hasViolations,
}) {
    return (
        <Box sx={{ mt: 4 }}>
            <Typography variant="h6" gutterBottom>
                Review & Submit
            </Typography>

            {/* Partner Details Section */}
            <Card variant="outlined" sx={{ mb: 2 }}>
                <CardContent>
                    <Typography variant="h6" gutterBottom color="primary">
                        Partner Information
                    </Typography>
                    {selectedPartner ? (
                        <Grid container spacing={2}>
                            <Grid item xs={12} md={6}>
                                <TextFieldComponent
                                    fullWidth
                                    label="Partner ID"
                                    value={selectedPartner.partner_id || "N/A"}
                                    disabled
                                />
                            </Grid>
                            <Grid item xs={12} md={6}>
                                <TextFieldComponent
                                    fullWidth
                                    label="Partner Name"
                                    value={selectedPartner.partner_name || "N/A"}
                                    disabled
                                />
                            </Grid>
 </Grid>
                    ) : (
                        <Typography color="error">No partner selected</Typography>
                    )}
                </CardContent>
            </Card>

            {/* Proposal Details Section */}
            <Card variant="outlined" sx={{ mb: 2 }}>
                <CardContent>
                    <Typography variant="h6" gutterBottom color="primary">
                        Proposal Details
                    </Typography>
                    <Grid container spacing={2}>
                        <Grid item md={6}>
                            <TextFieldComponent
                                label="Partial Proposal Name"
                                value={proposalName}
                                disabled
                            />
                        </Grid>
                        <Grid item md={6}>
                            <BasicDatePicker
                                label="Expiry Date"
                                placeholder="Select Expiry Date"
                                selectedDate={new Date(expiryDate)}
                                minDate={new Date(new Date().setDate(new Date().getDate() + 1))} // Tomorrow
                                onChange={(newVal) => {
                                    setExpiryDate(newVal);
                                }}
                            />
                        </Grid>
                    </Grid>
                </CardContent>
            </Card>

            {/* Selected Countries Table */}
            <Card variant="outlined" sx={{ mb: 3 }}>
                <CardContent>
                    <Typography variant="h6" gutterBottom color="primary">
                        Selected Countries & Services
                    </Typography>

                    <ItemsTables 
                        data={pricingData} 
                        readOnly 
                        showForecastFields={true}
                        getVelocityPeriods={() => []}
                        getVelocityTypes={() => []}
                        getRanges={() => []}
                        getCalcModes={() => []}
                        enablePagination={true}
                        itemsPerPage={25}
                    />
                </CardContent>
            </Card>

            {/* Initial comments Button */}
            {hasViolations && (
                <Box sx={{ mb: 3, mt: 3 }}>
                    <TextFieldComponent
                        fullWidth
                        multiline
                        rows={4}
                        height={"80px"}
                        label={`Justification for low rates.`}
                        placeholder={"Please provide a business justification for item rates lower than minimum rate."}
                        value={initialComments}
                        onChange={(e) => setInitialComments(e.target.value)}
                        variant="outlined"
                        error={!initialComments.trim()}
                        helperText={!initialComments.trim() ? "Please provide a business justification for item rates lower than minimum rate." : ""}
                        required
                    />
                </Box>
            )}
            {/* Submit Button */}
            <Box sx={{ display: "flex", justifyContent: "end" }}>
                <Button
                    variant="contained"
                    color="primary"
                    size="small"
                    onClick={handleSubmitProposal}
                    disabled={hasViolations ? !initialComments.trim() : false}
                >
                    Submit Proposal
                </Button>
            </Box>
            <Box sx={{ mb: 2 }}>
                {error && (
                    <Alert severity="error" sx={{ mb: 2 }}>
                        {error}
                    </Alert>
                )}
                {success && (
                    <Alert severity="success" sx={{ mb: 2 }}>
                        {success}
                    </Alert>
                )}
            </Box>
        </Box>
    );
}

export default Step3;

default .js 
export const feeValueOptions = [
    {
        label: "Proposed Fee",
        value: "proposed_fee",
        value2: "custom_proposed_fee",
    },
    {
        label: "Negotiation Level 1",
        value: "fee_negotiation_level_1",
        value2: "custom_fee_negotiation_level_1",
    },
    {
        label: "Negotiation Level 2",
        value: "fee_negotiation_level_2",
        value2: "custom_fee_negotiation_level_2",
    },
    {
        label: "Minimum Fee",
        value: "minimum_fee",
        value2: "custom_minimum_fee",
    },
];

import ReactQuill, { Quill } from "react-quill";
import "react-quill/dist/quill.snow.css"; // import Quill styles
import CustomToolBar from "@utils/customToobar";
import { useState, useRef } from "react";
import { Button, CircularProgress, Dialog, DialogActions, DialogContent, DialogTitle, FormHelperText, Stack } from "@mui/material";
import AttachmentIcon from "@mui/icons-material/Attachment";
import SendOutlinedIcon from "@mui/icons-material/SendOutlined";

import { Colors } from "@/theme/colors";
import TextFieldComponent from "@/common/TextField/TextFieldComponent";
import { validateEmail } from "@utils/validation/domainValidation";

const AlignStyle = Quill.import("attributors/style/align");
const BackgroundStyle = Quill.import("attributors/style/background");
const ColorStyle = Quill.import("attributors/style/color");
const DirectionStyle = Quill.import("attributors/style/direction");
const FontStyle = Quill.import("attributors/style/font");
const SizeStyle = Quill.import("attributors/style/size");
// SizeStyle.whitelist = ['12px', '14px', '16px', '18px', '20px', '24px', '32px'];

Quill.register(AlignStyle, true);
Quill.register(BackgroundStyle, true);
Quill.register(ColorStyle, true);
Quill.register(DirectionStyle, true);
Quill.register(FontStyle, true);
Quill.register(SizeStyle, true);

const modules = {
  toolbar: {
    container: "#toolbar",
  },
};
const formats = [
  "font",
  "size",
  "bold",
  "italic",
  "underline",
  "strike",
  "color",
  "background",
  "script",
  "headers",
  "blockquote",
  "code-block",
  "indent",
  "list",
  "direction",
  "align",
  "image",
];

function EmailModal({ open = true, onClose, emailData, setEmailData, emailError, setEmailError, sendEmail }) {
    const quillRef = useRef(null);
    const [htmlData, setHtmlData] = useState(emailData?.body || "");

    return (
        <div>
            {/* Send email proposal to partner modal */}
            <Dialog
                open={open}
                onClose={() => onClose(false)}
                maxWidth="md"
                slotProps={{
                    paper: {
                        sx: { overflowX: "hidden" },
                    },
                }}
                fullWidth
            >
                <DialogTitle bgcolor={Colors.tpBlue} color={Colors.light}>
                    Email Proposal to Partner: {emailData?.partner?.partner_name}
                </DialogTitle>
                <DialogContent>
                    <TextFieldComponent
                        fullWidth
                        label="Email *"
                        value={emailData.email}
                        sx={{ my: 2 }}
                        required
                        error={emailError.email}
                        helperText={emailError.email}
                        onChange={(e) => {
                            const invalid = validateEmail(e.target.value);
                            setEmailData((prev) => ({
                                ...prev,
                                email: e.target.value,
                            }));
                            if(invalid) {
                                setEmailError((prev) => ({ ...prev, email: invalid }));
                            } else {
                                setEmailError((prev) => ({ ...prev, email: "" }));
                            }
                        }}
                    />

                    <TextFieldComponent
                        fullWidth
                        label="CC"
                        value={emailData.cc}
                        error={emailError.cc}
                        onChange={(e) => {
                            setEmailData((prev) => ({
                                ...prev,
                                cc: e.target.value,
                            }));
                            const invalid = validateEmail(e.target.value);
                            if (e.target.value && invalid) {
                                setEmailError((prev) => ({ ...prev, cc: invalid }));
                            } else {
                                setEmailError((prev) => ({ ...prev, cc: "" }));
                            }
                        }}
                        sx={{ mb: 2 }}
                        placeholder="Enter comma-separated email addresses"
                        helperText={emailError.cc || "Optional: Separate multiple email addresses with commas"}
                    />

                    <TextFieldComponent
                        fullWidth
                        label="BCC"
                        value={emailData.bcc}
                        error={emailError.bcc}
                        onChange={(e) => {
                            setEmailData((prev) => ({
                                ...prev,
                                bcc: e.target.value,
                            }));
                            const invalid = validateEmail(e.target.value);
                            if (e.target.value && invalid) {
                                setEmailError((prev) => ({ ...prev, bcc: invalid }));
                            } else {
                                setEmailError((prev) => ({ ...prev, bcc: "" }));
                            }
                        }}
                        sx={{ mb: 2 }}
                        placeholder="Enter comma-separated email addresses"
                        helperText={emailError.bcc || "Optional: Separate multiple email addresses with commas"}
                    />

                    <TextFieldComponent
                        fullWidth
                        label="Subject"
                        value={emailData.subject}
                        onChange={(e) => {
                            setEmailData((prev) => ({
                                ...prev,
                                subject: e.target.value,
                            }));
                            if (!e.target.value) {
                                setEmailError((prev) => ({ ...prev, subject: "Subject is required" }));
                            } else {
                                setEmailError((prev) => ({ ...prev, subject: "" }));
                            }
                        }}
                        sx={{ mb: 2 }}
                        required
                        error={emailError.subject || !emailData.subject.trim()}
                        helperText={emailError.subject || !emailData.subject.trim() ? "Subject is required" : ""}
                    />
                    <div>
                        Body *
                        <CustomToolBar />
                        <ReactQuill
                            style={{ height: "350px", border: emailError?.body ? "1px solid red" : "" }}
                            ref={quillRef}
                            value={htmlData || emailData?.body}
                            modules={modules}
                            formats={formats}
                            onChange={(content, delta, source, editor) => {
                                const text = editor.getText();
                                setHtmlData(content);
                                setEmailData((prev) => ({
                                    ...prev,
                                    body: content,
                                }));
                                setEmailError((error) => ({
                                    ...error,
                                    body: text?.trim() ? "" : "Body is required",
                                }));
                            }}
                        />
                        {
                            emailError.body &&
                            <FormHelperText error>
                                { emailError.body }
                            </FormHelperText>
                        }
                    </div>

                    { /*<TextField
                        fullWidth
                        label="Body"
                        multiline
                        rows={6}
                        value={emailData.body}
                        onChange={(e) => {
                            setEmailData((prev) => ({
                                ...prev,
                                body: e.target.value,
                            }));
                            if (!e.target.value) {
                                setEmailError((prev) => ({ ...prev, body: "Body is required" }));
                            } else {
                                setEmailError((prev) => ({ ...prev, body: "" }));
                            }
                        }}
                        required
                        error={emailError.body || !emailData.body.trim()}
                        helperText={emailError.body || !emailData.body.trim() ? "Body is required" : ""}
                    /> */}
                </DialogContent>
                <DialogActions
                    sx={{
                        justifyContent:"space-between"
                    }}
                >
                    <Stack>
                        {emailData.attachments?.length > 0 &&
                         emailData.attachments.map((attachment, index) => {
                            if(!attachment?.filename || !attachment?.content) {
                                return null;
                            }
                            return (
                                  <Stack
                                      key={'attachment-' + index}
                                      direction="row"
                                      alignItems="center"
                                      gap={1}
                                      mx={1}
                                      px={1}
                                      sx={{ border: '1px solid #ccc', borderRadius: '5px'}}
                                  >
                                      {attachment?.filename}{" "}
                                      <AttachmentIcon style={{ color: Colors.tpBlue, marginBottom: '2px' }}/>
                                  </Stack>);
                        })}
                    </Stack>
                    <Stack direction="row" gap={2}>
                        <Button onClick={() => onClose(false)}>Cancel</Button>
                        <Button
                            variant="contained"
                            color="primary"
                            onClick={() => sendEmail()}
                            disabled={
                                emailData.isLoading ||
                                !emailData.subject.trim() ||
                                !emailData.body.trim() ||
                                (!emailData.email || validateEmail(emailData.email)) ||
                                (emailData.cc && validateEmail(emailData.cc)) ||
                                (emailData.bcc && validateEmail(emailData.bcc))
                              }
                          >
                            {emailData.isLoading
                              ? <CircularProgress size={20} />
                              : <SendOutlinedIcon sx={{ mr: 1 }} />}
                            {" "}Send
                        </Button>
                    </Stack>
                </DialogActions>
            </Dialog>

    </div>)
}

export default EmailModal

import React, { useState, useEffect, useMemo } from "react";
import {
    Dialog,
    DialogTitle,
    DialogContent,
    Typography,
    Table,
    TableBody,
    TableCell,
    TableContainer,
    TableHead,
    TableRow,
    Paper,
    Grid,
    Box,
    Button,
    Tabs,
    Tab,
    Accordion,
    AccordionSummary,
    AccordionDetails,
    Chip,
} from "@mui/material";
import { VisibilityOutlined, ExpandMore } from "@mui/icons-material";
import { useNavigate, useParams } from "react-router-dom";

import { request } from "@utils/index";
import {
    getProposalByID,
    getProposalHistoryByID,
    getProposalCombinedHistory,
} from "@services-(pms)/proposals";
import { Colors } from "@/theme/colors";
import { useSelector } from "react-redux";

import apiGateWay from "@utils/axiosGateWay";

// Utility function to convert to Title Case
export const toTitleCase = (str) => {
    if (!str) return "";
    return str
        .replace(/_/g, " ")
        .split(" ")
        .map((word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
        .join(" ");
};

const formatValue = (value) => {
    if (value === null || value === undefined) {
        return "No value";
    }

    // Check if it's a datetime string by looking for common patterns
    const datePattern = /^\d{4}-\d{2}-\d{2}(T|\s)\d{2}:\d{2}:\d{2}/;
    const dateOnlyPattern = /^\d{4}-\d{2}-\d{2}$/;

    if (typeof value === "string") {
        // Check if it's a date string
        if (datePattern.test(value) || dateOnlyPattern.test(value)) {
            try {
                const date = new Date(value);
                if (!isNaN(date.getTime())) {
                    return date.toLocaleString("en-US", {
                        year: "numeric",
                        month: "short",
                        day: "numeric",
                        hour: dateOnlyPattern.test(value) ? undefined : "2-digit",
                        minute: dateOnlyPattern.test(value) ? undefined : "2-digit",
                    });
                }
            } catch (e) {
                // If date parsing fails, treat as regular string
                console.warn("Failed to parse date:", value);
            }
        }
        // For non-date strings, apply title case
        return toTitleCase(value);
    }

    // Handle other types
    if (typeof value === "boolean") {
        return value ? "Yes" : "No";
    }

    if (typeof value === "number") {
        // Check if it's a whole number
        if (Number.isInteger(value)) {
            return value.toString();
        }
        // Format float to 2 decimal places
        return value.toFixed(2);
    }

    if (typeof value === "object") {
        return <NestedValueTable data={value} />;
    }

    // Fallback for any other type
    return String(value);
};

const NestedValueTable = ({ data }) => {
    // Determine if data is a complex object or JSON-like string
    const parseData = () => {
        if (typeof data !== "string") return data;

        try {
            // Try to parse if it looks like a JSON string
            return JSON.parse(data);
        } catch {
            // If parsing fails, return original string
            return data;
        }
    };

    const parsedData = parseData();

    // If it's a simple string or number, just return it
    if (typeof parsedData !== "object" || parsedData === null) {
        return <Typography>{String(parsedData)}</Typography>;
    }

    // If it's an array, handle it differently
    if (Array.isArray(parsedData)) {
        return (
            <TableContainer component={Paper}>
                <Table size="small">
                    <TableBody>
                        {parsedData.map((item, index) => (
                            <TableRow key={index}>
                                <TableCell>
                                    {typeof item === "object" && item !== null ? (
                                        <NestedValueTable data={JSON.stringify(item, null, 2)} />
                                    ) : (
                                        String(item)
                                    )}
                                </TableCell>
                            </TableRow>
                        ))}
                    </TableBody>
                </Table>
            </TableContainer>
        );
    }

    // If it's an object
    return (
        <TableContainer component={Paper}>
            <Table size="small">
                <TableHead
                    sx={{
                        "& .MuiTableCell-root": {
                            background: Colors.tpBlue,
                            color: Colors.light,
                        },
                    }}
                >
                    <TableRow>
                        <TableCell>Key</TableCell>
                        <TableCell>Value</TableCell>
                    </TableRow>
                </TableHead>
                <TableBody>
                    {Object.entries(parsedData).map(([key, value]) => (
                        <TableRow key={key}>
                            <TableCell>{toTitleCase(key)}</TableCell>
                            <TableCell>
                                {typeof value === "object" && value !== null ? (
                                    <NestedValueTable data={JSON.stringify(value, null, 2)} />
                                ) : typeof value === "string" &&
                                    key.toLowerCase().includes("status") ? (
                                    toTitleCase(String(value))
                                ) : (
                                    String(value)
                                )}
                            </TableCell>
                        </TableRow>
                    ))}
                </TableBody>
            </Table>
        </TableContainer>
    );
};

const ItemHistoryTable = ({ records, onShowDetails }) => {
    const userList = useSelector((state) => state.commonReducer.userList);

    const getUserName = (userId) => {
        return userList?.find((user) => user.account_id == userId)?.username;
    };

    return (
        <TableContainer component={Paper}>
            <Table>
                <TableHead
                    sx={{
                        bgcolor: Colors.tpBlue,
                        "& .MuiTableCell-root": {
                            color: Colors.light,
                            textAlign: "center",
                            bgcolor: Colors.tpBlue,
                        },
                    }}
                >
                    <TableRow>
                        <TableCell>Version</TableCell>
                        <TableCell>Change Type</TableCell>
                        <TableCell>Changed By</TableCell>
                        <TableCell>Changed At</TableCell>
                        <TableCell>Reason</TableCell>
                        {/* <TableCell>Details</TableCell> */}
                    </TableRow>
                </TableHead>
                <TableBody
                    sx={{
                        "& .MuiTableCell-root": {
                            textAlign: "center",
                            color: Colors.tpBlue,
                        },
                        "& .MuiTableCell-root.comments-cell": {
                            textAlign: "left",
                        },
                    }}
                >
                    {records.map((record) => (
                        <TableRow key={record.id}>
                            <TableCell>v{record.version}</TableCell>
                            <TableCell>{toTitleCase(record.change_type)}</TableCell>
                            <TableCell>{getUserName(record.changed_by)}</TableCell>
                            <TableCell>
                                {new Date(record.changed_at).toLocaleString()}
                            </TableCell>
                            <TableCell className="comments-cell">{record.change_reason}</TableCell>
                            {/* <TableCell>
                                {(record.old_data || record.new_data) && (
                                    <Typography
                                        color="primary"
                                        sx={{ cursor: "pointer" }}
                                        onClick={() => onShowDetails(record.old_data, record.new_data)}
                                    >
                                        View Changes
                                    </Typography>
                                )}
                            </TableCell> */}
                        </TableRow>
                    ))}
                </TableBody>
            </Table>
        </TableContainer>
    );
};

const ChangeDetailsDialog = ({ open, onClose, oldData, newData }) => {
    // Function to safely parse and flatten data
    const parseAndFlattenData = (data) => {
        if (!data) return {};

        const flattenObject = (obj, prefix = "") => {
            if (!obj || typeof obj !== "object") return {};

            return Object.keys(obj).reduce((acc, key) => {
                const prefixedKey = prefix ? `${prefix}.${key}` : key;

                if (
                    obj[key] &&
                    typeof obj[key] === "object" &&
                    !Array.isArray(obj[key])
                ) {
                    return {
                        ...acc,
                        ...flattenObject(obj[key], prefixedKey),
                    };
                }

                acc[prefixedKey] = obj[key];
                return acc;
            }, {});
        };

        // Flatten the object
        const flattened = flattenObject(data);
        // const flattened = data;
        return Object.keys(flattened).reduce((acc, key) => {
            acc[key] = flattened[key];
            return acc;
        }, {});
    };

    // Parse and flatten both old and new data
    const flatOldData = useMemo(() => parseAndFlattenData(oldData), [oldData]);
    const flatNewData = useMemo(() => parseAndFlattenData(newData), [newData]);

    // Combine keys from both old and new data
    const allKeys = useMemo(() => {
        const keys = new Set([
            ...Object.keys(flatOldData),
            ...Object.keys(flatNewData),
        ]);
        return Array.from(keys).sort();
    }, [flatOldData, flatNewData]);

    // Render a table for given data
    const renderDataTable = (data, title) => (
        <Box>
            <Typography variant="subtitle1" gutterBottom>
                {title}
            </Typography>
            <TableContainer component={Paper} sx={{ maxHeight: 400 }}>
                <Table stickyHeader>
                    <TableHead
                        sx={{
                            "& .MuiTableCell-root": {
                                background: Colors.tpBlue,
                                color: Colors.light,
                            },
                        }}
                    >
                        <TableRow>
                            <TableCell>Key</TableCell>
                            <TableCell>Value</TableCell>
                        </TableRow>
                    </TableHead>
                    <TableBody>
                        {allKeys.map((key) => (
                            <TableRow key={key}>
                                <TableCell
                                    sx={{
                                        fontWeight: "bold",
                                        wordBreak: "break-word",
                                    }}
                                >
                                    {toTitleCase(key)}
                                </TableCell>
                                <TableCell
                                    sx={{
                                        wordBreak: "break-words",
                                        whiteSpace: "pre-wrap",
                                        color:
                                            data[key] === undefined ? "text.secondary" : "inherit",
                                    }}
                                >
                                    {data[key] !== undefined
                                        ? formatValue(data[key])
                                        : "No value"}
                                </TableCell>
                            </TableRow>
                        ))}
                    </TableBody>
                </Table>
            </TableContainer>
        </Box>
    );

    return (
        <Dialog
            open={open}
            onClose={onClose}
            maxWidth="lg"
            fullWidth
            PaperProps={{
                sx: {
                    "& .MuiTypography-root": {
                        color: Colors.tpBlue,
                    },
                    "& .MuiTableCell-root": {
                        color: Colors.tpBlue,
                    },
                },
            }}
        >
            <DialogTitle sx={{ color: Colors.tpBlue }}>Change Details</DialogTitle>
            <DialogContent>
                <Grid container spacing={2}>
                    <Grid item xs={6}>
                        {renderDataTable(flatOldData, "Previous State")}
                    </Grid>
                    <Grid item xs={6}>
                        {renderDataTable(flatNewData, "New State")}
                    </Grid>
                </Grid>
            </DialogContent>
        </Dialog>
    );
};

export default function ProposalHistory() {
    const { id: proposalId } = useParams();
    const navigate = useNavigate();

    const userList = useSelector((state) => state.commonReducer.userList);

    const [proposalHistory, setProposalHistory] = useState([]);
    const [itemHistory, setItemHistory] = useState([]);
    const [error, setError] = useState("");
    const [loading, setLoading] = useState(true);
    const [activeTab, setActiveTab] = useState(0);
    const [detailsDialog, setDetailsDialog] = useState({
        open: false,
        oldData: null,
        newData: null,
    });

    useEffect(() => {
        fetchHistory();
    }, [proposalId]);

    const fetchHistory = async () => {
        try {
            const response = await request({
                api: getProposalCombinedHistory,
                params: { id: proposalId }
            });

            // Make sure to check the actual response structure from your backend
            setProposalHistory(response?.data?.data?.proposal_history || []);
            setItemHistory(response?.data?.data?.item_history || []);
            setLoading(false);
        } catch (error) {
            console.error("Error fetching history:", error);
            setError("Failed to fetch proposal history");
            setLoading(false);
        }
    };

    const showChangeDetails = (oldData, newData) => {
        setDetailsDialog({
            open: true,
            oldData: oldData || {},
            newData: newData || {},
        });
    };

    const getUserName = (userId) => {
        if (!userId) return "Unknown User";
        if (userId === "system") return "System";
        return userList?.find((user) => user.account_id == userId)?.username;
    };

    // Group item history by item_id for better organization
    const groupedItemHistory = useMemo(() => {
        const grouped = {};
        itemHistory.forEach(record => {
            if (!grouped[record.item_id]) {
                grouped[record.item_id] = [];
            }
            grouped[record.item_id].push(record);
        });
        return grouped;
    }, [itemHistory]);

    if (loading) {
        return (
            <Box sx={{ p: 3 }}>
                <Typography>Loading history...</Typography>
            </Box>
        );
    }

    return (
        <Box
            sx={{
                p: 3,
                color: Colors.tpBlue,
                "& .MuiTypography-root": {
                    color: Colors.tpBlue,
                },
                "& .MuiTableCell-root": {
                    color: Colors.tpBlue,
                },
            }}
        >
            <Button variant="text" onClick={() => navigate(-1)}>
                {"< "}Back
            </Button>
            <Typography variant="h4" gutterBottom sx={{ color: Colors.tpBlue }}>
                Proposal History
            </Typography>

            {error && (
                <Box sx={{ mb: 2 }}>
                    <Typography color="error">{error}</Typography>
                </Box>
            )}

            <Box sx={{ borderBottom: 1, borderColor: "divider", mb: 2 }}>
                <Tabs
                    value={activeTab}
                    onChange={(e, newValue) => setActiveTab(newValue)}
                    indicatorColor="primary"
                    textColor="primary"
                >
                    <Tab label="Proposal Changes" />
                    {/* <Tab label="Item Changes" /> */}
                </Tabs>
            </Box>

            {activeTab === 0 && (
                <Paper>
                    <Table>
                        <TableHead
                            sx={{
                                bgcolor: Colors.tpBlue,
                                "& .MuiTableCell-root": {
                                    color: Colors.light,
                                    textAlign: "center",
                                    bgcolor: Colors.tpBlue,
                                },
                            }}
                        >
                            <TableRow>
                                <TableCell>Version</TableCell>
                                <TableCell>Change Type</TableCell>
                                <TableCell>Changed By</TableCell>
                                <TableCell>Changed At</TableCell>
                                <TableCell>Comments</TableCell>
                                {/* <TableCell>Details</TableCell> */}
                            </TableRow>
                        </TableHead>
                        <TableBody
                            sx={{
                                "& .MuiTableCell-root": {
                                    textAlign: "center",
                                    color: Colors.tpBlue,
                                },
                            }}
                        >
                            {proposalHistory.map((record) => (
                                console.log({ record }) ||
                                <TableRow key={record.id}>
                                    <TableCell>v{record.version}</TableCell>
                                    <TableCell>{toTitleCase(record.change_type)}</TableCell>
                                    <TableCell>{getUserName(record.changed_by)}</TableCell>
                                    {/* <TableCell>User: {record.changed_by}</TableCell> */}
                                    <TableCell>
                                        {new Date(record.changed_at).toLocaleString()}
                                    </TableCell>
                                    <TableCell>{record.reason || record.change_reason}</TableCell>
                                    {/* <TableCell>
                                        {(record.old_data || record.new_data) && (
                                            <Typography
                                                color="primary"
                                                sx={{ cursor: "pointer" }}
                                                onClick={() =>
                                                    showChangeDetails(record.old_data, record.new_data)
                                                }
                                            >
                                                View Changes
                                            </Typography>
                                        )}
                                    </TableCell> */}
                                </TableRow>
                            ))}
                        </TableBody>
                    </Table>
                </Paper>
            )}

            {/* {activeTab === 1 && (
                <Box>
                    {Object.keys(groupedItemHistory).length > 0 ? (
                        Object.entries(groupedItemHistory).map(([itemId, records]) => (
                            <Accordion key={itemId} sx={{ mb: 2 }}>
                                <AccordionSummary
                                    expandIcon={<ExpandMore />}
                                    sx={{
                                        backgroundColor: Colors.light,
                                        borderBottom: `1px solid ${Colors.tpBorderColor || "#e0e0e0"}`,
                                    }}
                                >
                                    <Typography>
                                        Item ID: {itemId}
                                        {records[0]?.country_code ?
                                            ` - ${records[0].country_code}` :
                                            ''}
                                    </Typography>
                                </AccordionSummary>
                                <AccordionDetails>
                                    <ItemHistoryTable
                                        records={records}
                                        onShowDetails={showChangeDetails}
                                    />
                                </AccordionDetails>
                            </Accordion>
                        ))
                    ) : (
                        <Typography align="center" sx={{ mt: 4 }}>
                            No item history found for this proposal
                        </Typography>
                    )}
                </Box>
            )} */}

            <ChangeDetailsDialog
                open={detailsDialog.open}
                onClose={() =>
                    setDetailsDialog({ open: false, oldData: null, newData: null })
                }
                oldData={detailsDialog.oldData}
                newData={detailsDialog.newData}
            />
        </Box>
    );
}
import Dropdown from "@/common/Dropdown/Dropdown";
import VirtualTable from "@/common/TableMUI/VirtualTable";
import TextFieldComponent from "@/common/TextField/TextFieldComponent";
import { Colors } from "@/theme/colors";
import {
    feeTypeOptions,
    hardcodedCountries,
    velocityPeriodOptions,
    velocityTypeOptions,
} from "@modules-(onboarding)/configuration/FeeRackRate/FeeRackRate/defaults";
import DeleteIcon from "@mui/icons-material/Delete";
import WarningIcon from "@mui/icons-material/Warning";
import { Box, Checkbox, Grid2, IconButton, Stack, Tooltip, Typography } from "@mui/material";
import { addLoader, removeLoader } from "@redux/commonSlice";
import { getFeeRackSpeedTexts } from "@services/feeRackRate";
import { request } from "@utils/index";
import { useState, useEffect, useCallback, useMemo } from "react";
import { useDispatch } from "react-redux";
import { useSelector } from "react-redux";
import CurrencyInput from "react-currency-input-field";
import EditIcon from "@mui/icons-material/Edit";
import { feeValueOptions } from "./defaults";

const feeValueNewOptions = [...feeValueOptions, { label: "Custom Fee", value: "custom", isDisabled: true }];
const keyboardShortcuts = `Use shortcuts in keyboard to enter values \n
1k   =  1,000,
1.5k =  1,500,
1m   =  1,000,000
1b   =  1,000,000,000
... and so on`;

const prepareOptions = (row, data) => (item, index) => {
    let { label, ...rest } = item;
    if (data.type === "flat") {
        switch (rest.value) {
            case 'proposed_fee':
                label = `${label} - ${data.proposedFlat}`
                break;
            case 'minimum_fee':
                label = `${label} - ${data.minimumFlat}`
                break;
            case 'fee_negotiation_level_1':
                label = `${label} - ${data.neg1Flat}`
                break;
            case 'fee_negotiation_level_2':
                label = `${label} - ${data.neg2Flat}`
                break;
            default:
                break;
        }
    } else {
        switch (rest.value) {
            case 'proposed_fee':
                label = `${label} - ${data.proposedPercentage}`
                break;
            case 'minimum_fee':
                label = `${label} - ${data.minimumPercentage}`
                break;
            case 'fee_negotiation_level_1':
                label = `${label} - ${data.neg1Percentage}`
                break;
            case 'fee_negotiation_level_2':
                label = `${label} - ${data.neg2Percentage}`
                break;
            default:
                break;
        }
    }
    return { ...rest, label };
}

function ItemsTables({
    data = [],
    showActions = false,
    proposal_status,
    setData,
    getVelocityTypes,
    getVelocityPeriods,
    getRanges,
    getCalcModes,
    getTransactionTypeOptions,
    getCoverageOptions,
    readOnly,
    onEditVolume,
    highlightBelowMinimum = false,
    belowMinimumRows = [],
    swapSlabAndCommittedVolume = false,
    updateSlabBasedOnCommittedVolume,
    showForecastFields = false,
    showDeleteButton = true,
    showEditButton = false,
    onDeleteItem = null,
    deleteButtonTooltip = "Delete Item",
    enablePagination = false,
    itemsPerPage = 10,
    handleFeeValueApplyToAll,
}) {
    // Whether to show delete button - use the passed prop directly instead of considering readOnly
    // This allows delete button to be shown in View mode
    const shouldShowDeleteButton = showDeleteButton;
    const dispatch = useDispatch();

    const { countryList, masterValueList, fiList } = useSelector((state) => state.commonReducer);

    const countryName = useMemo(() => {
        const names = {};
        countryList.forEach((country) => {
            names[country.country_code] = country.country_name;
        });
        hardcodedCountries.forEach((country) => {
            names[country.value] = country.label;
        });
        return names;
    }, [countryList]);

    const masterData = useMemo(() => {
        return masterValueList.map((item) => ({
            label: item.master_value_name,
            value: item.master_value_value,
            master_value_type: item.master_value_type,
            master_value_display_order: item.master_value_display_order,
        }));
    }, [masterValueList]);

    const [paymentInstruments, setPaymentInstruments] = useState([]);

    // Helper function to parse minimum rates from custom_minimum_fee JSON
    const parseMinimumRates = (row) => {
        let minimumFlat = 0;
        let minimumPercentage = 0;

        const rackCustomMinimumFee = row?.custom_minimum_fee || row?.fee_rack_rate?.custom_minimum_fee;
        const rackMinimumFee = row?.minimum_fee || row?.fee_rack_rate?.minimum_fee;

        // For mixed rates, the structure is:
        // - Flat minimum is in the regular "minimum_fee" field
        // - Percentage minimum is in the "custom_minimum_fee" JSON
        if (rackCustomMinimumFee) {
            try {
                const customMinimum = JSON.parse(rackCustomMinimumFee);

                // Check for new structure: {"feeName":"CUSTOM_FEE","calculationMode":"percentage","fees":"7"}
                if (customMinimum.calculationMode && customMinimum.fees) {
                    const minValue = parseFloat(customMinimum.fees || 0);
                    if (customMinimum.calculationMode === 'percentage') {
                        // The JSON contains the percentage minimum
                        minimumPercentage = minValue;
                        // Get flat minimum from regular minimum_fee field
                        minimumFlat = parseFloat(rackMinimumFee || 0);
                    } else if (customMinimum.calculationMode === 'flat') {
                        // The JSON contains the flat minimum
                        minimumFlat = minValue;
                        // Try to get percentage minimum from other sources
                        minimumPercentage = parseFloat(row.minimum_fee_percentage || rackMinimumFee || 0);
                    }
                }
                // Check for legacy structure: {"flat_fee":"15","percentage_fee":"7"}
                else if (customMinimum.flat_fee || customMinimum.percentage_fee) {
                    minimumFlat = parseFloat(customMinimum.flat_fee || 0);
                    minimumPercentage = parseFloat(customMinimum.percentage_fee || 0);
                }
                else {
                    // Fallback to individual minimum fields
                    minimumFlat = parseFloat(rackMinimumFee || 0);
                    minimumPercentage = parseFloat(row.minimum_fee_percentage || 0);
                }
            } catch (e) {
                console.log("Failed to parse custom_minimum_fee JSON:", e);
                // Fallback to individual minimum fields if JSON parsing fails
                minimumFlat = parseFloat(rackMinimumFee || 0);
                minimumPercentage = parseFloat(row.minimum_fee_percentage || 0);
            }
        } else {
            // No custom minimum fee JSON, use individual fields
            minimumFlat = parseFloat(rackMinimumFee || 0);
            minimumPercentage = parseFloat(rackMinimumFee || 0);
        }

        return { minimumFlat, minimumPercentage };
    };

    // Helper function to parse minimum rates from custom_minimum_fee JSON
    const parseProposedRates = (row) => {
        let proposedFlat = 0;
        let proposedPercentage = 0;

        const rackCustomProposedFee = row?.custom_proposed_fee || row?.fee_rack_rate?.custom_proposed_fee;
        const rackProposedFee = row?.proposed_fee || row?.fee_rack_rate?.proposed_fee;

        const rackCustomMinimumFee = row?.custom_minimum_fee || row?.fee_rack_rate?.custom_minimum_fee;
        const rackMinimumFee = row?.minimum_fee || row?.fee_rack_rate?.minimum_fee;

        // For mixed rates, the structure is:
        // - Flat minimum is in the regular "minimum_fee" field
        // - Percentage minimum is in the "custom_minimum_fee" JSON
        if (rackCustomProposedFee) {
            try {
                const customProposed = JSON.parse(row.custom_proposed_fee);

                // Check for new structure: {"feeName":"CUSTOM_FEE","calculationMode":"percentage","fees":"7"}
                if (customProposed.calculationMode && customProposed.fees) {
                    const proposedValue = parseFloat(customProposed.fees || 0);
                    if (customProposed.calculationMode === 'percentage') {
                        // The JSON contains the percentage minimum
                        proposedPercentage = proposedValue;
                        // Get flat minimum from regular minimum_fee field
                        proposedFlat = parseFloat(rackProposedFee || 0);
                    } else if (customProposed.calculationMode === 'flat') {
                        // The JSON contains the flat minimum
                        proposedFlat = proposedValue;
                        // Try to get percentage minimum from other sources
                        proposedPercentage = parseFloat(rackProposedFee || 0);
                    }
                }
                // Check for legacy structure: {"flat_fee":"15","percentage_fee":"7"}
                else if (customProposed.flat_fee || customProposed.percentage_fee) {
                    proposedFlat = parseFloat(customProposed.flat_fee || 0);
                    proposedPercentage = parseFloat(customProposed.percentage_fee || 0);
                }
                else {
                    // Fallback to individual minimum fields
                    proposedFlat = parseFloat(rackProposedFee || 0);
                    proposedPercentage = parseFloat(row.minimum_fee_percentage || rackMinimumFee || 0);
                }
            } catch (e) {
                console.log("Failed to parse custom_minimum_fee JSON:", e);
                // Fallback to individual minimum fields if JSON parsing fails
                proposedFlat = parseFloat(rackProposedFee || 0);
                proposedPercentage = parseFloat(rackProposedFee || 0);
            }
        } else {
            // No custom minimum fee JSON, use individual fields
            proposedFlat = parseFloat(rackProposedFee || 0);
            proposedPercentage = parseFloat(rackProposedFee || 0);
        }

        return { proposedFlat, proposedPercentage };
    };

    // Helper function to parse minimum rates from custom_minimum_fee JSON
    const parseNeg1Rates = (row) => {
        let neg1Flat = 0;
        let neg1Percentage = 0;

        const rackCustomNeg1Fee = row?.custom_fee_negotiation_level_1 || row?.fee_rack_rate?.custom_fee_negotiation_level_1;
        const rackNeg1Fee = row?.fee_negotiation_level_1 || row?.fee_rack_rate?.fee_negotiation_level_1;

        const rackCustomMinimumFee = row?.custom_minimum_fee || row?.fee_rack_rate?.custom_minimum_fee;
        const rackMinimumFee = row?.minimum_fee || row?.fee_rack_rate?.minimum_fee;

        // For mixed rates, the structure is:
        // - Flat minimum is in the regular "minimum_fee" field
        // - Percentage minimum is in the "custom_minimum_fee" JSON
        if (rackCustomNeg1Fee) {
            try {
                const customNeg1 = JSON.parse(row.custom_fee_negotiation_level_1);

                // Check for new structure: {"feeName":"CUSTOM_FEE","calculationMode":"percentage","fees":"7"}
                if (customNeg1.calculationMode && customNeg1.fees) {
                    const neg1Value = parseFloat(customNeg1.fees || 0);
                    if (customNeg1.calculationMode === 'percentage') {
                        // The JSON contains the percentage minimum
                        neg1Percentage = neg1Value;
                        // Get flat minimum from regular minimum_fee field
                        neg1Flat = parseFloat(rackNeg1Fee || 0);
                    } else if (customNeg1.calculationMode === 'flat') {
                        // The JSON contains the flat minimum
                        neg1Flat = neg1Value;
                        // Try to get percentage minimum from other sources
                        neg1Percentage = parseFloat(rackNeg1Fee || 0);
                    }
                }
                // Check for legacy structure: {"flat_fee":"15","percentage_fee":"7"}
                else if (customNeg1.flat_fee || customNeg1.percentage_fee) {
                    neg1Flat = parseFloat(customNeg1.flat_fee || 0);
                    neg1Percentage = parseFloat(customNeg1.percentage_fee || 0);
                }
                else {
                    // Fallback to individual minimum fields
                    neg1Flat = parseFloat(rackMinimumFee || 0);
                    neg1Percentage = parseFloat(row.minimum_fee_percentage || rackMinimumFee || 0);
                }
            } catch (e) {
                console.log("Failed to parse custom_minimum_fee JSON:", e);
                // Fallback to individual minimum fields if JSON parsing fails
                neg1Flat = parseFloat(rackNeg1Fee || 0);
                neg1Percentage = parseFloat(rackNeg1Fee || 0);
            }
        } else {
            // No custom minimum fee JSON, use individual fields
            neg1Flat = parseFloat(rackNeg1Fee || 0);
            neg1Percentage = parseFloat(rackNeg1Fee || 0);
        }

        return { neg1Flat, neg1Percentage };
    };

    // Helper function to parse minimum rates from custom_minimum_fee JSON
    const parseNeg2Rates = (row) => {
        let neg2Flat = 0;
        let neg2Percentage = 0;

        const rackCustomNeg2Fee = row?.custom_fee_negotiation_level_2 || row?.fee_rack_rate?.custom_fee_negotiation_level_2;
        const rackNeg2Fee = row?.fee_negotiation_level_2 || row?.fee_rack_rate?.fee_negotiation_level_2;

        const rackCustomMinimumFee = row?.custom_minimum_fee || row?.fee_rack_rate?.custom_minimum_fee;
        const rackMinimumFee = row?.minimum_fee || row?.fee_rack_rate?.minimum_fee;

        // For mixed rates, the structure is:
        // - Flat minimum is in the regular "minimum_fee" field
        // - Percentage minimum is in the "custom_minimum_fee" JSON
        if (rackCustomNeg2Fee) {
            try {
                const customNeg2 = JSON.parse(row.custom_fee_negotiation_level_2);

                // Check for new structure: {"feeName":"CUSTOM_FEE","calculationMode":"percentage","fees":"7"}
                if (customNeg2.calculationMode && customNeg2.fees) {
                    const neg2Value = parseFloat(customNeg2.fees || 0);
                    if (customNeg2.calculationMode === 'percentage') {
                        // The JSON contains the percentage minimum
                        neg2Percentage = neg2Value;
                        // Get flat minimum from regular minimum_fee field
                        neg2Flat = parseFloat(rackNeg2Fee || 0);
                    } else if (customNeg2.calculationMode === 'flat') {
                        // The JSON contains the flat minimum
                        neg2Flat = neg2Value;
                        // Try to get percentage minimum from other sources
                        neg2Percentage = parseFloat(rackNeg2Fee || 0);
                    }
                }
                // Check for legacy structure: {"flat_fee":"15","percentage_fee":"7"}
                else if (customNeg2.flat_fee || customNeg2.percentage_fee) {
                    neg2Flat = parseFloat(customNeg2.flat_fee || 0);
                    neg2Percentage = parseFloat(customNeg2.percentage_fee || 0);
                }
                else {
                    // Fallback to individual minimum fields
                    neg2Flat = parseFloat(rackMinimumFee || 0);
                    neg2Percentage = parseFloat(row.minimum_fee_percentage || rackMinimumFee || 0);
                }
            } catch (e) {
                console.log("Failed to parse custom_minimum_fee JSON:", e);
                // Fallback to individual minimum fields if JSON parsing fails
                neg2Flat = parseFloat(rackNeg2Fee || 0);
                neg2Percentage = parseFloat(rackNeg2Fee || 0);
            }
        } else {
            // No custom minimum fee JSON, use individual fields
            neg2Flat = parseFloat(rackNeg2Fee || 0);
            neg2Percentage = parseFloat(rackNeg2Fee || 0);
        }

        return { neg2Flat, neg2Percentage };
    };

    useEffect(() => {
        const getSpeedTextAPICall = async () => {
            dispatch(addLoader("SPEED_TEXT"));
            try {
                const response = await request({
                    api: getFeeRackSpeedTexts,
                });
                if (response.status === 200) {
                    const paymentInstruments = response.data.data2?.map((item) => ({
                        label: item.type,
                        value: item.payment_instrument_id,
                        name: item.payment_instrument_name,
                        type: item.type?.toUpperCase(),
                    }));
                    setPaymentInstruments(paymentInstruments);
                }
            } finally {
                dispatch(removeLoader("SPEED_TEXT"));
            }
        };
        getSpeedTextAPICall();
    }, [dispatch]);

    const transactionTypeOptions = useMemo(() => {
        return masterData
            .filter((item) => item.master_value_type == "TRANSACTION_TYPE")
            .sort((b, a) => b.master_value_display_order - a.master_value_display_order);
    }, [masterData]);

    const getCountryName = (code) => {
        return countryName[code];
    }

    // Helper function to check if a rate is below minimum
    const isRateBelowMinimum = (row, index) => {
        console.log(`DEBUG - isRateBelowMinimum checking row (index: ${index}):`, row);
        console.log(`DEBUG - fee: ${row.fee}, min_flat: ${row.minimum_fee_flat}, fee_percentage: ${row.fee_percentage}, min_percentage: ${row.minimum_fee_percentage}`);

        if (belowMinimumRows.includes(index)) {
            console.log(`DEBUG - Row ${index} is in belowMinimumRows list`);
            return true;
        }

        // Case 1: Standard flat rate
        if ((row.fee_type === "FLAT" || row.calculation_mode === "flat") &&
            row.custom_fee_enable !== 1) {
            const feeValue = parseFloat(row.fee || 0);
            // Try to get minimum from different possible field names
            const minFee = parseFloat(row.minimum_fee_flat || row.minimum_fee || 0);
            console.log(`DEBUG - Checking flat rate: ${feeValue} < ${minFee}? ${feeValue < minFee}`);

            if (feeValue < minFee) {
                console.log(`DEBUG - Row ${index} has flat rate below minimum`);
                return true;
            }
        }

        // Case 2: Standard percentage rate
        if ((row.fee_type === "PERCENTAGE" || row.calculation_mode === "percentage") &&
            row.custom_fee_enable !== 1) {
            const feePercentage = parseFloat(row.fee_percentage || 0);
            // Try to get minimum from different possible field names
            const minPercentage = parseFloat(row.minimum_fee_percentage || row.minimum_fee || 0);
            console.log(`DEBUG - Checking percentage rate: ${feePercentage} < ${minPercentage}? ${feePercentage < minPercentage}`);

            if (feePercentage < minPercentage) {
                console.log(`DEBUG - Row ${index} has percentage rate below minimum`);
                return true;
            }
        }

        // Case 3: Mixed rate - Component-wise validation
        if ((row.fee_type === "MIXED" || row.custom_fee_enable === 1)) {
            console.log(`DEBUG - Checking mixed rate for row ${index}`);

            // Parse minimum rates using helper function
            const { minimumFlat, minimumPercentage } = parseMinimumRates(row);
            console.log(`DEBUG - Parsed minimum rates: flat=${minimumFlat}, percentage=${minimumPercentage}`);

            // Get current proposed rates for both components
            const proposedFlat = parseFloat(row.fee_flat || 0);
            const proposedPercentage = parseFloat(row.fee_percentage || 0);

            console.log(`DEBUG - Mixed rate comparison: proposed flat=${proposedFlat} vs min=${minimumFlat}, proposed percentage=${proposedPercentage} vs min=${minimumPercentage}`);

            // Both components must be >= minimum for the rate to NOT be below minimum
            const flatAcceptable = proposedFlat >= minimumFlat;
            const percentageAcceptable = proposedPercentage >= minimumPercentage;
            const isBelowMinimum = !(flatAcceptable && percentageAcceptable);

            console.log(`DEBUG - Mixed rate validation: flat acceptable=${flatAcceptable}, percentage acceptable=${percentageAcceptable}, below minimum=${isBelowMinimum}`);

            if (isBelowMinimum) {
                console.log(`DEBUG - Row ${index} mixed rate is below minimum`);
                return true;
            }

            console.log(`DEBUG - Row ${index} mixed rate meets minimum requirements`);
            return false;
        }

        // Check if the row has been explicitly marked as below minimum
        if (row.isRateBelowMinimum === true) {
            console.log(`DEBUG - Row ${index} is explicitly marked as below minimum`);
            return true;
        }

        console.log(`DEBUG - Row ${index} is NOT below minimum`);
        return false;
    };

    // First, let's modify ItemsTables.jsx to better handle different delete scenarios
    const handleDeleteItem = (row, index) => {
        console.log('Delete clicked', row, index);

        if (onDeleteItem) {
            // Use custom delete handler if provided (for View page)
            onDeleteItem(row, index);
        } else {
            // Default behavior (for Generate page)
            setData((prev) => {
                const newData = [...prev];
                newData.splice(index, 1);
                return newData;
            });
        }
    };

    // Auto-populate the appropriate slab range based on committed volume
    const handleCommittedVolumeChange = useCallback((index, value) => {
        if (updateSlabBasedOnCommittedVolume && !readOnly) {
            // Use a timeout to prevent too many updates at once
            setTimeout(() => {
                updateSlabBasedOnCommittedVolume(index, value);
            }, 300);
        }
    }, [updateSlabBasedOnCommittedVolume, readOnly]);

    const handleSelectAll = useCallback((e) => {
        const checkedData = JSON.parse(JSON.stringify(data || [])).map((r) => ({ ...r, check_box: e.target.checked }));
        setData(checkedData);
    }, [data, setData]);

    const handleSelectRow = useCallback((e, index) => {
        const newData = [...data];
        newData[index].check_box = e.target.checked;
        setData(newData);
    }, [data, setData]);

    const coverageOptions = useCallback((countryCode, payout_type) => {
        const newArray = [
            {
                value: "All",
                label: "All",
            },
        ];

        // get the fi list for the country
        let bb = fiList?.filter((x) => x?.dest_country_code === countryCode);

        console.log("DEBUG - FI List From DB ", fiList)

        // get the payment instrument based on payout type
        const payout = paymentInstruments?.find((pi) => pi.value === payout_type);
        console.log("DEBUG - paymentInstruments From DB ", paymentInstruments)

        bb = bb?.filter((x) => {
            // if payout type is all will get to see all options
            const instruments = payout?.type?.toUpperCase();
            if (instruments?.includes("ALL")) {
                return true;
            }

            return instruments?.includes(x?.type?.toUpperCase());
        });

        if (bb?.length) {
            newArray.push(
                ...bb.map((item) => ({
                    value: item.fi_code,
                    label: item.fi_name,
                }))
            );
        }

        return newArray;
    }, [fiList, paymentInstruments]);

    const selectAll = useMemo(() => {
        return data.every((row) => row?.check_box === true);
    }, [data]);

    const someSelected = useMemo(() => {
        return data.some((row) => row?.check_box === true);
    }, [data]);

    // Define the slab range column
    const slabRangeColumn = {
        headerName: "Slab Range",
        field: "range",
        width: "180px",
        minWidth: "180px",
        renderCell: ({ row, index }) => {
            // console.log('RENDERING FEE COLUMN!!!!!', { row, index });
            if (readOnly) {
                const minRange = row?.min_value || 0;
                const maxRange = row?.max_value || 999999999;
                return (
                    <div
                        dangerouslySetInnerHTML={{
                            __html: `${minRange} - ${Number(maxRange) == 999999999 ? "∞" : maxRange}`,
                        }}
                    ></div>
                );
            }
            const values = getRanges(row);
            if (values?.length > 0) {
                const options = values.map((d) => ({
                    label: `${d.min_value} - ${Number(d.max_value) == 999999999 ? "∞" : d.max_value
                        }`,
                    value: d.id,
                    ...d,
                }));
                const v = options.find((d) => d.value == row.fee_rack_rate_id);

                if (values.length > 0 && !row.fee_rack_rate_id && !readOnly) {
                    // We need to do this synchronously 
                    setTimeout(() => {
                        const firstRange = options[0];
                        setData?.((prev) => {
                            const newData = [...prev];
                            if (!newData[index]?.fee_rack_rate_id) {
                                newData[index] = {
                                    ...newData[index],
                                    fee_rack_rate_id: firstRange?.value,
                                    min_value: firstRange?.otherData?.min_value,
                                    max_value: firstRange?.otherData?.max_value,
                                    // Ensure fee is properly set to prevent "below minimum rate" warning
                                    fee: firstRange?.otherData?.proposed_fee || firstRange?.otherData?.fee,
                                    fee_percentage: firstRange?.otherData?.fee_percentage,
                                    minimum_fee_flat: firstRange?.otherData?.minimum_fee_flat,
                                    minimum_fee_percentage: firstRange?.otherData?.minimum_fee_percentage,
                                    fee_type: firstRange?.otherData?.fee_type,
                                    calculation_mode: firstRange?.otherData?.calculation_mode || firstRange?.otherData?.fee_type?.toLowerCase()
                                };

                                // Set committed volume based on the selected range
                                // For 0-infinity or any range with infinity, use 999999999
                                const committedValue = firstRange?.otherData?.max_value === 999999999
                                    ? 999999999
                                    : firstRange?.otherData?.max_value;
                                newData[index].commited_volume = committedValue;
                                newData[index].committed_volume = committedValue; // Both spellings for backend
                            }
                            return newData;
                        });
                    }, 0);
                }

                return (
                    <Dropdown
                        name={"slab" + index}
                        options={options}
                        selectedOption={v || null}
                        onChange={(e, val) => {
                            setData?.((prev) => {
                                const newData = [...prev];
                                newData[index] = {
                                    ...newData[index],
                                    fee_rack_rate_id: val?.value,
                                    min_value: val?.otherData?.min_value,
                                    max_value: val?.otherData?.max_value,
                                    // Ensure fee is properly set to prevent "below minimum rate" warning
                                    fee: val?.otherData?.proposed_fee || val?.otherData?.fee,
                                    fee_percentage: val?.otherData?.fee_percentage,
                                    minimum_fee_flat: val?.otherData?.minimum_fee_flat,
                                    minimum_fee_percentage: val?.otherData?.minimum_fee_percentage,
                                    fee_type: val?.otherData?.fee_type,
                                    calculation_mode: val?.otherData?.calculation_mode || val?.otherData?.fee_type?.toLowerCase(),
                                    minimum_fee: val?.otherData?.minimum_fee,
                                    custom_minimum_fee: val?.otherData?.custom_minimum_fee,
                                };

                                // Auto-update committed volume when range changes
                                // For 0-infinity or any range with infinity, use 999999999
                                const committedValue = val?.otherData?.max_value === 999999999
                                    ? 999999999
                                    : val?.otherData?.max_value;
                                newData[index].commited_volume = committedValue;
                                newData[index].committed_volume = committedValue; // Both spellings for backend

                                return newData;
                            });
                        }}
                    />
                );
            }
            if (data[index])
                data[index]['fee_rack_rate_id'] == row.frr_id;
            return (
                <div
                    dangerouslySetInnerHTML={{
                        __html: `0 - ∞`,
                    }}
                ></div>
            );
        },
    };

    // Define the committed volume column
    const committedVolumeColumn = {
        headerName: <div title={keyboardShortcuts}>Committed Volume</div>,
        field: "committed_volume",
        width: "200px",
        maxWidth: "200px",
        renderCell: ({ row, index }) => {
            // In readOnly mode, show as plain text
            if (readOnly) {
                const volume = row.commited_volume || row.committed_volume || 0;
                return (
                    <Box sx={{ padding: 1 }}>
                        <Typography variant="body2">
                            {Number(volume).toLocaleString()}
                        </Typography>
                    </Box>
                );
            }

            return (
                <CurrencyInput
                    customInput={TextFieldComponent}
                    disabled={readOnly}
                    name={`FEE_${index}`}
                    value={row.commited_volume}
                    decimalScale={0}
                    allowDecimals={false}
                    onValueChange={(value) => {
                        setData?.((prev) => {
                            // Both spellings for backend
                            const intValue = parseInt(value || 0, 10);
                            prev[index].commited_volume = intValue;
                            prev[index].committed_volume = intValue;
                            return [...prev];
                        });
                        // Trigger slab range update when committed volume changes
                        handleCommittedVolumeChange(index, value);
                    }}
                />
            );
        },
    };

    // Basic columns that always appear in the same order
    const baseColumns = [
        {
            width: readOnly ? "0px" : "5px",
            headerName: <>
                {!readOnly && <Checkbox
                    name="select_all"
                    id="select-all-items"
                    checked={selectAll || null}
                    indeterminate={!selectAll && someSelected}
                    onChange={handleSelectAll}
                />}
            </>,
            field: "check_box",
            renderCell: ({ row, index }) => {
                return !readOnly && <Checkbox
                    name={`select_item_${index}`}
                    id={`select-item-${index}`}
                    checked={row.check_box || null}
                    onChange={(e) => handleSelectRow(e, index)}
                />;
            }
        },
        {
            width: "20px",
            headerName: "Sr. No.",
            field: "sr_no",
            renderCell: ({ row, index }) => {
                const rowIndex = data.findIndex(item => item.id === row.id) || index;
                return rowIndex + 1;
            },
        },
        {
            width: "100px",
            minWidth: "100px",
            headerName: "Country",
            field: "country_code",
            renderCell: ({ row }) => {
                const code = row?.country_code;
                const countryName = getCountryName(code);
                return (
                    <Box>
                        {countryName} ({code})
                    </Box>
                );
            },
        },
        {
            // width: "80px",
            // maxWidth: "120px",
            headerName: <>Payout<br />currency</>,
            field: "payout_currency",
            renderCell: ({ row }) => {
                const currency = row?.payout_currency;
                return <Box>{currency}</Box>;
            },
        },

        {
            headerName: "Payout Type",
            field: "payout_type",
            width: "180px",
            maxWidth: "180px",
            renderCell: ({ row }) => {
                const payout_type = row?.payout_type;
                const paymentInstrument = paymentInstruments?.find(
                    (item) => item.value == payout_type
                );

                return <Box>{paymentInstrument?.label}</Box>;
            },
        },
        {
            headerName: "Coverage",
            field: "coverage",
            width: "180px",
            maxWidth: "180px",
            renderCell: ({ row, index }) => {
                const options = coverageOptions(row?.country_code, row?.payout_type);
                const coverage = options?.find((item) => item.value == row.coverage);

                return <Box>{coverage?.label}</Box>;
            },
        },
        {
            headerName: <>Transaction<br /> Type</>,
            field: "transaction_type",
            width: "180px",
            maxWidth: "180px",
            renderCell: ({ row, index }) => {
                const transactionType = transactionTypeOptions?.find(
                    (item) => item.value == row.transaction_type
                );

                if (readOnly) {
                    return <Box>{transactionType?.label}</Box>;
                }

                const availableTransactionTypeValues = getTransactionTypeOptions(row);
                const allTransactionTypeOptions = transactionTypeOptions?.map((item) => {
                    const transactionTypeOption = availableTransactionTypeValues?.find(
                        (option) => option.transaction_type == item.value
                    );
                    return {
                        value: item.value,
                        label: item.label,
                        frr_id: transactionTypeOption?.frr_id,
                    };
                });

                return (
                    <Dropdown
                        name="transaction_type"
                        options={allTransactionTypeOptions}
                        selectedOption={transactionType || null}
                        onChange={(e, v) => {
                            setData?.((prev) => {
                                prev[index].transaction_type = v?.value || null;
                                prev[index].fee_rack_rate_id = v?.frr_id || prev[index].fee_rack_rate_id;
                                return [...prev];
                            });
                        }}
                    />
                );
            },
        },

        {
            headerName: "Velocity Type",
            field: "velocity_type",
            width: "160px",
            minWidth: "160px",
            renderCell: ({ row, index }) => {
                const velocity = row.velocity_type;
                const velocity_type = velocityTypeOptions?.find(
                    (v) => v.value == (velocity || 2)
                );
                if (readOnly) {
                    return <Box>{velocity_type?.label}</Box>;
                }
                const velocity_types = getVelocityTypes(row, index);
                const options = velocityTypeOptions?.filter((v) =>
                    velocity_types?.includes(v.value)
                );
                if (options?.length > 0)
                    return (
                        <Dropdown
                            name={"velocity_type" + index}
                            options={options}
                            selectedOption={velocity_type || null}
                            onChange={(e, v) => {
                                setData?.((prev) => {
                                    prev[index].velocity_type = v?.value || null;
                                    return [...prev];
                                });
                            }}
                        />
                    );
                if (data[index]) {
                    data[index]['velocity_type'] = options?.[0]?.value || velocity;
                }
                return <Box>{velocity_type?.label || "Amount Based"}</Box>;
            },
        },
        {
            headerName: "Velocity Period",
            field: "velocity_period",
            width: "160px",
            minWidth: "160px",
            renderCell: ({ row, index }) => {
                const period = row.velocity_period;
                const v = velocityPeriodOptions?.find((v) => v.value == (period || 2));
                if (readOnly) {
                    return <Box>{v?.label || ""}</Box>;
                }
                const velocityPeriods = getVelocityPeriods?.(row, index)?.map(c => c.toString());
                const options = velocityPeriodOptions?.filter((v) =>
                    velocityPeriods?.includes(v.value?.toString())
                );
                if (options?.length > 0)
                    return (
                        <Dropdown
                            name={"velocity_period" + index}
                            options={options}
                            selectedOption={v || null}
                            onChange={(e, val) => {
                                setData?.((prev) => {
                                    prev[index].velocity_period = val?.value || null;
                                    return [...prev];
                                });
                            }}
                        />
                    );
                if (data[index]) {
                    data[index]['velocity_period'] = options?.[0]?.value || period;
                }
                return <Box>{v?.label || ""}</Box>;
            },
        },
        {
            headerName: "Fee Type",
            field: "fee_type",
            width: "120px",
            minWidth: "120px",
            renderCell: ({ row, index }) => {
                // In read-only mode, just show "Mixed" or the calculation mode label
                if (readOnly) {
                    if (row.custom_fee_enable === 1 || row.fee_type === "MIXED") {
                        return <Box>Mixed</Box>;
                    } else {
                        const calcMode = row.calculation_mode || "flat";
                        const option = feeTypeOptions.find(opt => opt.value === calcMode);
                        return <Box>{option?.label || calcMode}</Box>;
                    }
                }

                // In edit mode, handle dropdown options
                const calc = row.calculation_mode || "flat";
                const selectedOption = feeTypeOptions.find(opt => opt.value === calc);

                // Get available calculation modes for this item
                // const calcModes = getCalcModes(row, index);

                // Check if mixed option already exists
                const hasMixedOption = feeTypeOptions.some(mode => mode.value === "mixed");

                // Add mixed option if needed
                const optionsWithMixed = hasMixedOption ? feeTypeOptions : [
                    ...feeTypeOptions,
                    { label: "Mixed", value: "mixed" }
                ];

                // Determine the currently selected option
                const currentSelection = (row.custom_fee_enable === 1 || row.fee_type === "MIXED")
                    ? { label: "Mixed", value: "mixed" }
                    : selectedOption || null;

                return (
                    <Dropdown
                        name={`fee_type_${index}`}
                        options={optionsWithMixed}
                        selectedOption={currentSelection}
                        onChange={(e, val) => {
                            setData?.((prev) => {
                                if (val?.value === "mixed") {
                                    // Setting to mixed mode
                                    prev[index].custom_fee_enable = 1;
                                    prev[index].fee_type = "MIXED";
                                    // Keep existing calculation_mode for primary fee
                                } else {
                                    // Setting to single fee type
                                    prev[index].calculation_mode = val?.value || null;
                                    prev[index].custom_fee_enable = 0;
                                    prev[index].fee_type = val?.value?.toUpperCase() || "FLAT";

                                    // Clear secondary fee component
                                    prev[index].custom_proposed_fee = null;
                                    if (val?.value === "percentage") {
                                        prev[index].fee_flat = null;
                                    } else {
                                        prev[index].fee_percentage = null;
                                    }
                                }
                                const { minimum_fee, minimum_fee_percentage, min_value, max_value, ...otherPrev } = (prev?.[index] || {});
                                prev[index] = otherPrev
                                return [...prev];
                            });
                        }}
                    />
                );
            },
        },
    ];

    // Fee column that appears after slab range and committed volume
    const feeColumn = [{
        headerName: "Fee (in USD)",
        field: "fee",
        width: "200px",
        minWidth: "200px",
        renderCell: ({ row, index }) => {
            // Check if this rate is below minimum
            const isBelowMin = isRateBelowMinimum(row, index);

            // Check if this is a mixed rate (custom_fee_enable = 1)
            const isMixedRate = row.custom_fee_enable === 1 || row.fee_type === "MIXED";

            // In read-only mode, display the fee as plain text
            if (readOnly) {
                if (isMixedRate) {
                    // For mixed rates, try multiple data sources to get both components
                    let flatComponent = 0;
                    let percentageComponent = 0;

                    // Method 1: Direct fields (preferred for user-edited values)
                    if (row.fee_flat !== undefined && row.fee_flat !== null && row.fee_flat !== '') {
                        flatComponent = parseFloat(row.fee_flat);
                    }
                    if (row.fee_percentage !== undefined && row.fee_percentage !== null && row.fee_percentage !== '') {
                        percentageComponent = parseFloat(row.fee_percentage);
                    }

                    // Method 2: If either component is missing, try extracting from fee/JSON data
                    if (flatComponent === 0 || percentageComponent === 0) {
                        console.log(`Missing component for ${row.country_code}: flat=${flatComponent}, percentage=${percentageComponent}, trying fee/JSON extraction`);

                        // Extract primary component from fee field
                        let primaryValue = parseFloat(row.fee || 0);
                        let secondaryValue = 0;

                        // Parse secondary component from JSON
                        if (row.custom_proposed_fee) {
                            try {
                                const customFeeData = JSON.parse(row.custom_proposed_fee);
                                secondaryValue = parseFloat(customFeeData.fees || 0);
                            } catch (e) {
                                console.log("Failed to parse custom fee JSON", e);
                            }
                        }

                        // Map components based on calculation_mode and fill missing values
                        const calculationMode = row.calculation_mode || 'flat';

                        if (calculationMode === 'percentage') {
                            // Percentage is primary, flat is secondary
                            if (percentageComponent === 0) percentageComponent = primaryValue;
                            if (flatComponent === 0) flatComponent = secondaryValue;
                        } else {
                            // Flat is primary, percentage is secondary
                            if (flatComponent === 0) flatComponent = primaryValue;
                            if (percentageComponent === 0) percentageComponent = secondaryValue;
                        }
                    }

                    console.log(`Mixed rate display for ${row.country_code}: flat=${flatComponent}, percentage=${percentageComponent} (calculation_mode: ${row.calculation_mode})`);

                    return (
                        <Box sx={{ padding: 1 }}>
                            <Typography variant="body2">
                                {flatComponent.toFixed(2)} USD + {percentageComponent.toFixed(2)}%
                            </Typography>
                        </Box>
                    );
                } else {
                    // Single rate display
                    const calculationMode = row.calculation_mode || row.fee_type?.toLowerCase() || 'flat';
                    if (calculationMode === 'percentage') {
                        const percentageFee = parseFloat(row.fee_percentage || row.fee || 0);
                        return (
                            <Box sx={{ padding: 1 }}>
                                <Typography variant="body2">
                                    {percentageFee.toFixed(2)}%
                                </Typography>
                            </Box>
                        );
                    } else {
                        const flatFee = parseFloat(row.fee_flat || row.fee || 0);
                        return (
                            <Box sx={{ padding: 1 }}>
                                <Typography variant="body2">
                                    {flatFee.toFixed(2)} USD
                                </Typography>
                            </Box>
                        );
                    }
                }
            }

            // Render mixed rate inputs
            if (isMixedRate) {
                return (
                    <Stack direction="column" spacing={0.5}>
                        <Grid2 container spacing={1}>
                            {/* Primary Fee Input */}
                            {row.calculation_mode === "percentage" ? (
                                <Grid2 size={5.8}
                                    sx={{
                                        display: 'flex',
                                        alignItems: 'center',
                                        border: '1.8px solid #909bb8',
                                        borderRadius: '4px',
                                        backgroundColor: readOnly ? '#FBFCFD' : '#ffffff',
                                        '&:hover': {
                                            borderColor: !readOnly ? Colors.tpBlue : '#909bb8'
                                        },
                                        '&:focus-within': {
                                            borderColor: !readOnly ? Colors.tpBlue : '#909bb8'
                                        }
                                    }}
                                >
                                    <CurrencyInput
                                        customInput={TextFieldComponent}
                                        disabled={readOnly}
                                        name={`FEE_${index}`}
                                        value={row.fee_percentage}
                                        decimalsLimit={2}
                                        onValueChange={(value) => {
                                            setData?.((prev) => {
                                                const updatedData = [...prev];
                                                const rowData = updatedData[index];
                                                rowData.fee_percentage = value;
                                                rowData.fee = value; // Also update fee for consistency in percentage-primary mode
                                                rowData.userEdited = true; // Mark as user edited

                                                // For mixed rates, validate both components
                                                if (rowData.custom_fee_enable === 1 || rowData.fee_type === "MIXED") {
                                                    // Parse minimum rates using helper function
                                                    const { minimumFlat, minimumPercentage } = parseMinimumRates(rowData);

                                                    const proposedFlat = parseFloat(rowData.fee_flat || 0);
                                                    const proposedPercentage = parseFloat(value || 0);
                                                    const flatAcceptable = proposedFlat >= minimumFlat;
                                                    const percentageAcceptable = proposedPercentage >= minimumPercentage;

                                                    rowData.isRateBelowMinimum = !(flatAcceptable && percentageAcceptable);
                                                }

                                                return updatedData;
                                            });
                                        }}
                                        style={{
                                            flex: 1,
                                            '& .MuiOutlinedInput-notchedOutline': {
                                                border: 'none'
                                            }
                                        }}
                                        inputProps={{
                                            style: {
                                                border: 'none',
                                                outline: 'none'
                                            }
                                        }}
                                    />
                                    <Typography
                                        variant="body2"
                                        sx={{
                                            px: 1,
                                            color: Colors.tpBlue,
                                            fontWeight: 600,
                                            backgroundColor: '#f5f5f5',
                                            borderLeft: '1px solid #909bb8',
                                            display: 'flex',
                                            alignItems: 'center',
                                            height: '100%',
                                            minHeight: '36px'
                                        }}
                                    >
                                        %
                                    </Typography>
                                </Grid2>
                            ) : (
                                <Grid2 size={6.2}>
                                    <CurrencyInput
                                        customInput={TextFieldComponent}
                                        disabled={readOnly}
                                        name={`FEE_${index}`}
                                        value={row.fee}
                                        decimalsLimit={2}
                                        onValueChange={(value) => {
                                            setData?.((prev) => {
                                                const updatedData = [...prev];
                                                const rowData = updatedData[index];
                                                rowData.fee = value;
                                                rowData.fee_flat = value; // Also update fee_flat for consistency
                                                rowData.userEdited = true; // Mark as user edited

                                                // For mixed rates, validate both components
                                                if (rowData.custom_fee_enable === 1 || rowData.fee_type === "MIXED") {
                                                    // Parse minimum rates using helper function
                                                    const { minimumFlat, minimumPercentage } = parseMinimumRates(rowData);

                                                    const proposedFlat = parseFloat(value || 0);
                                                    const proposedPercentage = parseFloat(rowData.fee_percentage || 0);
                                                    const flatAcceptable = proposedFlat >= minimumFlat;
                                                    const percentageAcceptable = proposedPercentage >= minimumPercentage;

                                                    rowData.isRateBelowMinimum = !(flatAcceptable && percentageAcceptable);
                                                }

                                                return updatedData;
                                            });
                                        }}
                                    />
                                </Grid2>
                            )}

                            {/* Secondary Fee Input */}
                            {row.calculation_mode !== "percentage" ? (
                                <Grid2 size={5.8}
                                    sx={{
                                        display: 'flex',
                                        alignItems: 'center',
                                        border: '1.8px solid #909bb8',
                                        borderRadius: '4px',
                                        backgroundColor: readOnly ? '#FBFCFD' : '#ffffff',
                                        '&:hover': {
                                            borderColor: !readOnly ? Colors.tpBlue : '#909bb8'
                                        },
                                        '&:focus-within': {
                                            borderColor: !readOnly ? Colors.tpBlue : '#909bb8'
                                        }
                                    }}
                                >
                                    <CurrencyInput
                                        customInput={TextFieldComponent}
                                        disabled={readOnly}
                                        name={`SECONDARY_FEE_${index}`}
                                        value={row.fee_percentage || 0}
                                        decimalsLimit={2}
                                        onValueChange={(value) => {
                                            setData?.((prev) => {
                                                const rowData = prev[index];
                                                rowData.fee_percentage = value;
                                                rowData.custom_fee_enable = 1;
                                                rowData.fee_type = "MIXED";
                                                rowData.userEdited = true; // Mark as user edited

                                                // For mixed rates, validate both components
                                                if (rowData.custom_fee_enable === 1 || rowData.fee_type === "MIXED") {
                                                    // Parse minimum rates using helper function
                                                    const { minimumFlat, minimumPercentage } = parseMinimumRates(rowData);

                                                    const proposedFlat = parseFloat(rowData.fee_flat || rowData.fee || 0);
                                                    const proposedPercentage = parseFloat(value || 0);
                                                    const flatAcceptable = proposedFlat >= minimumFlat;
                                                    const percentageAcceptable = proposedPercentage >= minimumPercentage;

                                                    rowData.isRateBelowMinimum = !(flatAcceptable && percentageAcceptable);
                                                }

                                                return [...prev];
                                            });
                                        }}
                                        style={{
                                            flex: 1,
                                            '& .MuiOutlinedInput-notchedOutline': {
                                                border: 'none'
                                            }
                                        }}
                                        inputProps={{
                                            style: {
                                                border: 'none',
                                                outline: 'none'
                                            }
                                        }}
                                    />
                                    <Typography
                                        variant="body2"
                                        sx={{
                                            px: 1,
                                            color: Colors.tpBlue,
                                            fontWeight: 600,
                                            backgroundColor: '#f5f5f5',
                                            borderLeft: '1px solid #909bb8',
                                            display: 'flex',
                                            alignItems: 'center',
                                            height: '100%',
                                            minHeight: '36px'
                                        }}
                                    >
                                        %
                                    </Typography>
                                </Grid2>
                            ) : (
                                <Grid2 size={6.2}>
                                    <CurrencyInput
                                        customInput={TextFieldComponent}
                                        disabled={readOnly}
                                        name={`SECONDARY_FEE_${index}`}
                                        value={row.fee_flat || 0}
                                        decimalsLimit={2}
                                        onValueChange={(value) => {
                                            setData?.((prev) => {
                                                const rowData = prev[index];
                                                rowData.fee_flat = value;
                                                rowData.custom_fee_enable = 1;
                                                rowData.fee_type = "MIXED";
                                                rowData.userEdited = true; // Mark as user edited

                                                // For mixed rates, validate both components
                                                if (rowData.custom_fee_enable === 1 || rowData.fee_type === "MIXED") {
                                                    // Parse minimum rates using helper function
                                                    const { minimumFlat, minimumPercentage } = parseMinimumRates(rowData);

                                                    const proposedFlat = parseFloat(value || 0);
                                                    const proposedPercentage = parseFloat(rowData.fee_percentage || rowData.fee || 0);
                                                    const flatAcceptable = proposedFlat >= minimumFlat;
                                                    const percentageAcceptable = proposedPercentage >= minimumPercentage;

                                                    rowData.isRateBelowMinimum = !(flatAcceptable && percentageAcceptable);
                                                }

                                                return [...prev];
                                            });
                                        }}
                                    />
                                </Grid2>
                            )}
                        </Grid2>

                        {/* Warning indicator for below minimum rates - Now shown for mixed rates too */}
                        {(isBelowMin || (highlightBelowMinimum && isRateBelowMinimum(row, index))) && (
                            <Typography variant="caption" sx={{ color: Colors.tpRed, fontSize: "0.7rem" }}>
                                <Tooltip title={isMixedRate ? "One or more rate components are below minimum" : "Rate is below the minimum allowed rate"}>
                                    <Box sx={{ display: 'flex', alignItems: 'center', gap: 0.5 }}>
                                        <WarningIcon sx={{ fontSize: '0.9rem' }} />
                                        <span>Below minimum rate</span>
                                    </Box>
                                </Tooltip>
                            </Typography>
                        )}
                    </Stack>
                );
            }

            // Render single rate inputs
            return (
                <Stack direction="column" spacing={0.5}>
                    {row.calculation_mode === "percentage" ? (
                        <Box
                            sx={{
                                display: 'flex',
                                alignItems: 'center',
                                border: `1.8px solid ${isBelowMin ? Colors.tpRed : '#909bb8'}`,
                                borderRadius: '4px',
                                backgroundColor: readOnly ? '#FBFCFD' : '#ffffff',
                                '&:hover': {
                                    borderColor: readOnly ? (isBelowMin ? Colors.tpRed : '#909bb8') : Colors.tpBlue
                                },
                                '&:focus-within': {
                                    borderColor: readOnly ? (isBelowMin ? Colors.tpRed : '#909bb8') : Colors.tpBlue
                                }
                            }}
                        >
                            <CurrencyInput
                                customInput={TextFieldComponent}
                                disabled={readOnly}
                                name={`FEE_${index}`}
                                value={row.fee_percentage}
                                decimalsLimit={2}
                                onValueChange={(value) => {
                                    setData?.((prev) => {
                                        const updatedData = [...prev];
                                        const rowData = updatedData[index];
                                        rowData.fee_percentage = value;
                                        rowData.userEdited = true; // Mark as user edited

                                        // Skip below-minimum check for mixed rates
                                        if (rowData.custom_fee_enable !== 1 && rowData.fee_type !== "MIXED") {
                                            const numValue = parseFloat(value || 0);
                                            const minValue = parseFloat(rowData.minimum_fee_percentage || rowData.minimum_fee || 0);
                                            rowData.isRateBelowMinimum = numValue < minValue;
                                        }
                                        return updatedData;
                                    });
                                }}
                                style={{
                                    flex: 1,
                                    '& .MuiOutlinedInput-notchedOutline': {
                                        border: 'none'
                                    }
                                }}
                                inputProps={{
                                    style: {
                                        border: 'none',
                                        outline: 'none',
                                        color: isBelowMin ? Colors.tpRed : 'inherit'
                                    }
                                }}
                            />
                            <Typography
                                variant="body2"
                                sx={{
                                    px: 1,
                                    color: Colors.tpBlue,
                                    fontWeight: 600,
                                    backgroundColor: '#f5f5f5',
                                    borderLeft: '1px solid #909bb8',
                                    display: 'flex',
                                    alignItems: 'center',
                                    height: '100%',
                                    minHeight: '36px'
                                }}
                            >
                                %
                            </Typography>
                        </Box>
                    ) : (
                        <CurrencyInput
                            customInput={TextFieldComponent}
                            disabled={readOnly}
                            name={`FEE_${index}`}
                            value={row.fee}
                            sx={{
                                ...(isBelowMin && {
                                    input: {
                                        color: Colors.tpRed,
                                        borderColor: Colors.tpRed
                                    },
                                    fieldset: {
                                        borderColor: `${Colors.tpRed} !important`
                                    }
                                })
                            }}
                            decimalsLimit={2}
                            onValueChange={(value) => {
                                setData?.((prev) => {
                                    const updatedData = [...prev];
                                    const rowData = updatedData[index];
                                    rowData.fee = value;
                                    rowData.userEdited = true; // Mark as user edited

                                    // Skip below-minimum check for mixed rates
                                    if (rowData.custom_fee_enable !== 1 && rowData.fee_type !== "MIXED") {
                                        const numValue = parseFloat(value || 0);
                                        const minValue = parseFloat(rowData.minimum_fee_flat || rowData.minimum_fee || 0);
                                        rowData.isRateBelowMinimum = numValue < minValue;
                                    }
                                    return updatedData;
                                });
                            }}
                        />
                    )}

                    {/* Warning indicator for below minimum rates */}
                    {(isBelowMin || (highlightBelowMinimum && isRateBelowMinimum(row, index))) && (
                        <Typography variant="caption" sx={{ color: Colors.tpRed, fontSize: "0.7rem" }}>
                            <Tooltip title="Rate is below the minimum allowed rate">
                                <Box sx={{ display: 'flex', alignItems: 'center', gap: 0.5 }}>
                                    <WarningIcon sx={{ fontSize: '0.9rem' }} />
                                    <span>Below minimum rate</span>
                                </Box>
                            </Tooltip>
                        </Typography>
                    )}
                </Stack>
            );
        },
    },
    {
        headerName: "Fee Level",
        field: "fee_level",
        width: "300px",
        minWidth: "300px",
        renderCell: ({ row, index }) => {
            const { proposedFlat, proposedPercentage } = parseProposedRates(row);
            const { neg1Flat, neg1Percentage } = parseNeg1Rates(row);
            const { neg2Flat, neg2Percentage } = parseNeg2Rates(row);
            const { minimumFlat, minimumPercentage } = parseMinimumRates(row);

            console.log("row", row);
            console.log("proposed Fee Rates", proposedFlat, proposedPercentage);
            console.log("neg1 Fee Rates", neg1Flat, neg1Percentage);
            console.log("neg2 Fee Rates", neg2Flat, neg2Percentage);
            console.log("minimum Fee Rates", minimumFlat, minimumPercentage);

            let fee_level = "Custom Fee";
            let fee_level_custom = "";
            if (row.fee_type === "MIXED" || row.custom_fee_enable == 1) {
                fee_level_custom = "Custom Fee";
                if (row.calculation_mode == 'percentage') {
                    if (row.fee_percentage == proposedPercentage) {
                        fee_level = "Proposed Fee";
                    } else if (row.fee_percentage == neg1Percentage) {
                        fee_level = "Negotiation Level 1";
                    } else if (row.fee_percentage == neg2Percentage) {
                        fee_level = "Negotiation Level 2";
                    } else if (row.fee_percentage == minimumPercentage) {
                        fee_level = "Minimum Fee";
                    }

                    if (row.fee_flat == proposedFlat) {
                        fee_level_custom = "Proposed Fee";
                    } else if (row.fee_flat == neg1Flat) {
                        fee_level_custom = "Negotiation Level 1";
                    } else if (row.fee_flat == neg2Flat) {
                        fee_level_custom = "Negotiation Level 2";
                    } else if (row.fee_flat == minimumFlat) {
                        fee_level_custom = "Minimum Fee";
                    }
                } else {
                    if (row.fee_flat == proposedFlat) {
                        fee_level = "Proposed Fee";
                    } else if (row.fee_flat == neg1Flat) {
                        fee_level = "Negotiation Level 1";
                    } else if (row.fee_flat == neg2Flat) {
                        fee_level = "Negotiation Level 2";
                    } else if (row.fee_flat == minimumFlat) {
                        fee_level = "Minimum Fee";
                    }

                    if (row.fee_percentage == proposedPercentage) {
                        fee_level_custom = "Proposed Fee";
                    } else if (row.fee_percentage == neg1Percentage) {
                        fee_level_custom = "Negotiation Level 1";
                    } else if (row.fee_percentage == neg2Percentage) {
                        fee_level_custom = "Negotiation Level 2";
                    } else if (row.fee_percentage == minimumPercentage) {
                        fee_level_custom = "Minimum Fee";
                    }

                }
            } else if (row.fee_type === 'FLAT' || row.calculation_mode == 'flat') {
                if (row.fee == proposedFlat) {
                    fee_level = "Proposed Fee";
                } else if (row.fee == neg1Flat) {
                    fee_level = "Negotiation Level 1";
                } else if (row.fee == neg2Flat) {
                    fee_level = "Negotiation Level 2";
                } else if (row.fee == minimumFlat) {
                    fee_level = "Minimum Fee";
                }
            } else {
                if (row.fee_percentage == proposedPercentage) {
                    fee_level = "Proposed Fee";
                } else if (row.fee_percentage == neg1Percentage) {
                    fee_level = "Negotiation Level 1";
                } else if (row.fee_percentage == neg2Percentage) {
                    fee_level = "Negotiation Level 2";
                } else if (row.fee_percentage == minimumPercentage) {
                    fee_level = "Minimum Fee";
                }
            }
            if (readOnly) {
                return (
                    <Typography variant="body2" sx={{ wordBreak: "break-word", whiteSpace: "pre-wrap" }}>
                        {fee_level} {fee_level_custom ? `+ ${fee_level_custom}` : ''}
                    </Typography>
                );
            } else {
                if (row.fee_type === 'MIXED' || row.custom_fee_enable == 1) {
                    const value = feeValueNewOptions.find(fv => fv.label === fee_level);
                    const customValue = feeValueNewOptions.find(fv => fv.label === fee_level_custom);
                    return (
                        <Box display='flex' gap={1}>
                            <Dropdown
                                showTooltip
                                name={"fee_value"}
                                width="150px"
                                options={feeValueNewOptions.map(prepareOptions(row, { proposedFlat, proposedPercentage, neg1Flat, neg1Percentage, neg2Flat, neg2Percentage, minimumFlat, minimumPercentage, type: row.calculation_mode == 'percentage' ? 'percentage' : 'flat' }))}
                                selectedOption={value || null}
                                onChange={(e, v) => {
                                    // setFeeValue(v);
                                    handleFeeValueApplyToAll(v, index, true, row.calculation_mode == 'percentage' ? 'percentage' : 'flat');
                                }}
                            />
                            <Dropdown
                                showTooltip
                                name="custom_fee_value"
                                width="150px"
                                options={feeValueNewOptions.map(prepareOptions(row, { proposedFlat, proposedPercentage, neg1Flat, neg1Percentage, neg2Flat, neg2Percentage, minimumFlat, minimumPercentage, type: row.calculation_mode == 'percentage' ? 'flat' : 'percentage' }))}
                                selectedOption={customValue || null}
                                onChange={(e, v) => {
                                    handleFeeValueApplyToAll(v, index, true, row.calculation_mode == 'percentage' ? 'flat' : 'percentage');
                                    // setCustomFeeValue(v);
                                }}
                            />
                        </Box>
                    );
                } else {
                    return (
                        <Dropdown
                            showTooltip
                            name={"fee_value"}
                            width="150px"
                            options={feeValueNewOptions.map(prepareOptions(row, { proposedFlat, proposedPercentage, neg1Flat, neg1Percentage, neg2Flat, neg2Percentage, minimumFlat, minimumPercentage, type: row.calculation_mode == 'percentage' ? 'percentage' : 'flat' }))}
                            selectedOption={feeValueNewOptions.find(fv => fv.label === fee_level) || null}
                            onChange={(e, v) => {
                                handleFeeValueApplyToAll(v, index, true, row.calculation_mode);
                                // setFeeValue(v);
                            }}
                        />
                    );
                }
            }
        },
    }];

    // New columns for deal sheet - to be shown during proposal generation only!
    const forecastedTxnCountYear1 = {
        headerName: <>Forecasted<br />Transaction Count<br />1-12 Months</>,
        field: "forecasted_txn_count_year1",
        width: "180px",
        minWidth: "180px",
        renderCell: ({ row, index }) => {
            return (
                <CurrencyInput
                    customInput={TextFieldComponent}
                    disabled={readOnly}
                    name={`FORECASTED_TXN_YEAR1_${index}`}
                    value={row.forecasted_txn_count_year1 || ''}
                    decimalsLimit={0}
                    onValueChange={(value) => {
                        setData?.((prev) => {
                            prev[index].forecasted_txn_count_year1 = value;
                            return [...prev];
                        });
                    }}
                />
            );
        },
    };

    const forecastedTxnCountYear2 = {
        headerName: <>Forecasted<br />Transaction Count<br />13-24 Months</>,
        field: "forecasted_txn_count_year2",
        width: "180px",
        minWidth: "180px",
        renderCell: ({ row, index }) => {
            return (
                <CurrencyInput
                    customInput={TextFieldComponent}
                    disabled={readOnly}
                    name={`FORECASTED_TXN_YEAR2_${index}`}
                    value={row.forecasted_txn_count_year2 || ''}
                    decimalsLimit={0}
                    onValueChange={(value) => {
                        setData?.((prev) => {
                            prev[index].forecasted_txn_count_year2 = value;
                            return [...prev];
                        });
                    }}
                />
            );
        },
    };

    const forecastedTxnCountYear3 = {
        headerName: <>Forecasted<br />Transaction Count<br />25-36 Months</>,
        field: "forecasted_txn_count_year3",
        width: "180px",
        minWidth: "180px",
        renderCell: ({ row, index }) => {
            return (
                <CurrencyInput
                    customInput={TextFieldComponent}
                    disabled={readOnly}
                    name={`FORECASTED_TXN_YEAR3_${index}`}
                    value={row.forecasted_txn_count_year3 || ''}
                    decimalsLimit={0}
                    onValueChange={(value) => {
                        setData?.((prev) => {
                            prev[index].forecasted_txn_count_year3 = value;
                            return [...prev];
                        });
                    }}
                />
            );
        },
    };

    const averageTicketSize = {
        headerName: <>Average<br />Ticket Size</>,
        field: "average_ticket_size",
        width: "150px",
        minWidth: "150px",
        renderCell: ({ row, index }) => {
            return (
                <CurrencyInput
                    customInput={TextFieldComponent}
                    disabled={readOnly}
                    name={`AVG_TICKET_SIZE_${index}`}
                    value={row.average_ticket_size || ''}
                    decimalsLimit={2}
                    onValueChange={(value) => {
                        setData?.((prev) => {
                            prev[index].average_ticket_size = value;
                            return [...prev];
                        });
                    }}
                />
            );
        },
    };

    const forexPercentage = {
        headerName: "Forex %",
        field: "forex_percentage",
        width: "120px",
        minWidth: "120px",
        renderCell: ({ row, index }) => {
            return (
                <CurrencyInput
                    customInput={TextFieldComponent}
                    disabled={readOnly}
                    name={`FOREX_PERCENTAGE_${index}`}
                    value={row.forex_percentage || ''}
                    decimalsLimit={2}
                    prefix=""
                    onValueChange={(value) => {
                        // Ensure value doesn't exceed 100%
                        let validValue = value;
                        if (parseFloat(value) > 100) {
                            validValue = "100";
                        }

                        setData?.((prev) => {
                            prev[index].forex_percentage = validValue;
                            return [...prev];
                        });
                    }}
                />
            );
        },
    };

    const actionColumn = [
        {
            headerName: "Action",
            field: "action",
            align: 'center',
            renderCell: ({ row, index }) => {
                // console.log('Rendering action column', { row, index });
                return (
                    <Stack direction="row" justifyContent={'center'} alignItems={'center'}>
                        {showEditButton && (
                            <IconButton
                                title="Edit Volume"
                                onClick={(e) => {
                                    e.stopPropagation();
                                    console.log('Edit clicked', row);
                                    onEditVolume?.(row);
                                }}
                            >
                                <EditIcon color="primary" />
                            </IconButton>
                        )}

                        {shouldShowDeleteButton && (
                            <IconButton
                                title={deleteButtonTooltip}
                                onClick={(e) => {
                                    e.stopPropagation();
                                    console.log('Delete clicked', row);
                                    if (onDeleteItem) {
                                        onDeleteItem(row, index);
                                    } else {
                                        setData((prev) => {
                                            prev.splice(index, 1);
                                            return [...prev];
                                        });
                                    }
                                }}
                            >
                                <DeleteIcon color="primary" />
                            </IconButton>
                        )}
                    </Stack>
                );
            },
        },
    ];

    // Determine the columns to display based on showForecastFields prop
    const forecastColumns = showForecastFields ? [
        forecastedTxnCountYear1,
        forecastedTxnCountYear2,
        forecastedTxnCountYear3,
        averageTicketSize,
        forexPercentage
    ] : [];

    // Modify the columnsInOrder array to conditionally include the forecast columns
    const columnsInOrder = swapSlabAndCommittedVolume
        ? [
            ...baseColumns,
            committedVolumeColumn,
            slabRangeColumn,
            ...feeColumn,
            ...forecastColumns
        ]
        : [
            ...baseColumns,
            slabRangeColumn,
            committedVolumeColumn,
            ...feeColumn,
            ...forecastColumns
        ];

    console.log("Data Items", data);
    return (
        <>
            <VirtualTable
                getRowId={(row) => row.id}
                columns={/* Show actions column whenever showActions is true, regardless of readOnly state */
                    (!showActions && readOnly) ? [...columnsInOrder] : [...columnsInOrder, ...actionColumn]}
                rows={data}
                totalItemsCount={data.length}
                showPagination={enablePagination}
                numberOfItemsPerPage={enablePagination ? itemsPerPage : Number.POSITIVE_INFINITY}
                styles={{ minHeight: data?.length > 5 ? "760px" : "450px" }}
                options={{ scrollY: true }}
                overscan={{
                    main: 10,
                    reverse: 10,
                }}
                getRowClassName={(params) => {
                    const rowIndex = data.findIndex(row => row.id === params.id);
                    const isBelowMin = isRateBelowMinimum(params.row, rowIndex);
                    const isInList = belowMinimumRows.includes(rowIndex);
                    if (isBelowMin || isInList) {
                        return "below-minimum-rate-row";
                    }
                    return "";
                }}
                customCss={`
          .below-minimum-rate-row {
            background-color: #ffd5d2 !important; /* Stronger red color */
            // color: white !important;
          }
          .below-minimum-rate-row:hover {
            background-color: #ffd5d2 !important; /* Darker red on hover */
            // color: white !important;
          }
          .below-minimum-rate-row td {
            // color: white !important;
          }
        `}
            />
        </>
    );
}

export default ItemsTables;

import React, { useState, useEffect, useMemo } from "react";
import { useNavigate } from "react-router-dom";
import { useDispatch, useSelector } from "react-redux";
import { getItemsByProposalId, sendCustomEmailAPI } from "@services-(pms)/proposals";
import { getFeeRackSpeedTexts } from "@services/feeRackRate";
import KeyboardArrowDownIcon from "@mui/icons-material/KeyboardArrowDown";
import KeyboardArrowUpIcon from "@mui/icons-material/KeyboardArrowUp";
import RefreshIcon from "@mui/icons-material/Refresh";
import {
    Box,
    Card,
    CardHeader,
    Button,
    Typography,
    Alert,
    Stack,
    Collapse,
    IconButton,
    CircularProgress,
    Table,
    TableHead,
    TableBody,
    TableRow,
    TableCell,
    TableContainer,
    Paper
} from "@mui/material";
import { alpha } from "@mui/material/styles";

import { request } from "@utils/index";
import {
    getAllProposalAPI,
    updateProposalStatusAPI,
} from "@services-(pms)/proposals";
import { generateProposalPDF } from "@modules-(pms)/utils/pdfGenerator";
import { Colors } from "@/theme/colors";
import CustomModal from "@modules-(pms)/components/CustomModal";
import FilterHeader from "@/common/FilterHeader";
import useFilter from "@/hooks/useAppFilter";
import TextFieldComponent from "@/common/TextField/TextFieldComponent";
import { validateEmail } from "@utils/validation/domainValidation";
import { velocityPeriodOptions, velocityTypeOptions } from "@modules-(onboarding)/configuration/FeeRackRate/FeeRackRate/defaults";
import { CommonExcelDownload } from "@utils/excelGenerator";

import Pagination from "@mui/material/Pagination";
import { renderToaster } from "@utils/notificationUtils";
import { addLoader, removeLoader } from "@redux/commonSlice";
import EmailModal from "./EmailModal";
import { sortAndMapArray } from "@utils/helperFunctions";

const SORT_DIRECTION = {
    ASCENDING: "asc",
    DESCENDING: "desc",
};

const PROPOSAL_SORT_CONFIG = {
    field: "created_at",
    direction: SORT_DIRECTION.DESCENDING,
};

export default function Proposals({ access }) {
    const navigate = useNavigate();
    const dispatch = useDispatch();

    const countryList = useSelector((state) => state.commonReducer?.countryList);
    const currencyList = useSelector((state) => state.commonReducer?.currencyList);
    const partnerList = useSelector((state) => state.commonReducer?.partnerList);
    const masterValueList = useSelector((state) => state.commonReducer?.masterValueList);

    const PROPOSAL_STATUS_OPTIONS = useMemo(() => {
        const proposalStatus = masterValueList.filter((item) => item.master_value_type === 'PROPOSAL_STATUS');
        return sortAndMapArray(
            proposalStatus,
            "master_value_display_order",
              {
                  valueProperty: "master_value_value",
                  labelProperty1: "master_value_name",
              }
        );
    }, [masterValueList]);

    const [expandedRows, setExpandedRows] = useState(new Set());
    const [loadedItems, setLoadedItems] = useState({});

    const { filter } = useFilter();

    const [paymentInstruments, setPaymentInstruments] = useState([]);

    const [proposals, setProposals] = useState([]);
    const [limit, setLimit] = useState(10);
    const [dialogState, setDialogState] = useState({
        open: false,
        type: null,
        proposalId: null,
        comments: "",
    });

    const [page, setPage] = useState(1);
    const [totalPages, setTotalPages] = useState(1);

    // Initialize from URL params if available
    useEffect(() => {
        const params = new URLSearchParams(window.location.search);
        const pageParam = params.get('page');
        const limitParam = params.get('limit');

        if (pageParam) {
            const pageValue = parseInt(pageParam, 10);
            if (!isNaN(pageValue) && pageValue > 0) {
                setPage(pageValue);
            }
        }

        if (limitParam) {
            const limitValue = parseInt(limitParam, 10);
            if (!isNaN(limitValue) && limitValue > 0) {
                setLimit(limitValue);
            }
        }
    }, []);

    const [error, setError] = useState("");
    const [loading, setLoading] = useState(true);
    const [partnerNames, setPartnerNames] = useState({});

    useEffect(() => {
        const names = {};
        partnerList?.forEach((partner) => {
            names[partner.partner_id] = partner.partner_name;
        });
        setPartnerNames(names);
    }, [partnerList]);

    const filteredProposals = useMemo(() => {
        const tableRows = proposals.filter((row) => {
            return Object.entries(filter).every(([key, array]) => {
                if (array.length > 0) {
                    return array?.map(({ value }) => value).includes(row[key]);
                }
                return true;
            });
        });
        return tableRows;
    }, [filter, proposals]);

    useEffect(() => {
        if (filteredProposals) {
            setTotalPages(Math.ceil(filteredProposals.length / limit));
            // Reset to page 1 when filters change
            setPage(1);
        }
    }, [filteredProposals, limit]);

    const paginatedProposals = useMemo(() => {
        const startIndex = (page - 1) * limit;
        const endIndex = startIndex + limit;
        return filteredProposals.slice(startIndex, endIndex);
    }, [filteredProposals, page, limit]);

    const handlePageChange = (event, newPage) => {
        setPage(newPage);
        // Clear expanded rows when changing pages to avoid stale data
        setExpandedRows(new Set());

        // Update URL with new page number
        try {
            const params = new URLSearchParams(window.location.search);
            params.set('page', newPage);

            // Keep the limit if it exists in the URL
            if (!params.has('limit') && limit !== 10) {
                params.set('limit', limit);
            }

            const newUrl = `${window.location.pathname}?${params.toString()}`;
            window.history.replaceState({}, '', newUrl);
        } catch (error) {
            console.error("Error updating URL:", error);
        }
    };

    // Send proposal email to partner
    const [emailModal, setEmailModal] = useState(false);
    const [emailData, setEmailData] = useState({
        email: "",
        cc: "",
        bcc: "",
        subject: "Proposal - TerraPay",
        body: "",
        attachments: [],
        partner: null,
        proposal_id: null,
        isLoading: false,
    });
    const [emailError, setEmailError] = useState({
        email: "",
        cc: "",
        bcc: "",
        subject: "",
        body: "",
    });

    function titleCaseSp(str) {
        if (!str) return '';
        const strValue = String(str);
        return strValue
            .toLowerCase()
            .split(' ')
            .map(word => word.charAt(0).toUpperCase() + word.slice(1))
            .join(' ');
    }

    function titleCase_(str) {
        if (!str) return '';
        const strValue = String(str);
        return strValue
            .toLowerCase()
            .split('_')
            .map(word => word.charAt(0).toUpperCase() + word.slice(1))
            .join(' ');
    }

    const transactionTypeOptions = useMemo(() => {
        return masterValueList
            .filter((item) => item.master_value_type == "TRANSACTION_TYPE")
            .sort((b, a) => b.master_value_display_order - a.master_value_display_order);
    }, [masterValueList]);


    const getPayoutTypeName = (typeId) => {
        if (!typeId) return 'N/A';

        const instrument = paymentInstruments.find(item =>
            item.value.toString() === typeId.toString()
        );

        if (instrument) {
            return titleCase_(instrument.label || instrument.name);
        }

        // Fallback if mapping not found
        return `Type ${typeId}`;
    };

    const getVelocityTypeName = (typeId) => {
        if (typeId === null || typeId === undefined) {
            return 'N/A';
        }
        return velocityTypeOptions.find(v => v.value == typeId)?.label;
    };

    const getVelocityPeriodName = (type) => {
        if (!type) {
            return 'N/A';
        }
        return velocityPeriodOptions.find(v => v.value == type)?.label;
    }

    const getVolumeSlabName = (min,max) => {
        let minVal = min ?? "0";
        let maxVal = max ?? "Infinity";
        return `${minVal} - ${maxVal >= 999999999 ? "Infinity" : maxVal}`;
    }

    useEffect(() => {
        const getSpeedTextAPICall = async () => {
            try {
                const response = await request({
                    api: getFeeRackSpeedTexts,
                });
                if (response.status === 200) {
                    const instruments = response.data.data2?.map((item) => ({
                        label: item.type,
                        value: item.payment_instrument_id,
                        name: item.payment_instrument_name,
                        type: item.type?.toUpperCase(),
                    }));
                    setPaymentInstruments(instruments);
                }
            } catch (error) {
                console.error("Error fetching payment instruments:", error);
            }
        };

        getSpeedTextAPICall();
    }, []);

    const getCountryDisplayName = (countryCode) => {
        try {
            // Try to get the full country name
            // const name = getName ? getCountryName(countryCode) : countryCode;
            const name = countryList.find((c) => c.country_code == countryCode)?.country_name;

            if (!name) {
                // Handle special cases
                switch (countryCode) {
                    case 'AN': return 'Netherlands Antilles';
                    case 'XK': return 'Kosovo';
                    case 'IV': return "Côte d'Ivoire";
                    case '77': return 'SEPA';
                    default: return countryCode;
                }
            }
            return name;
        } catch (error) {
            console.warn(`Could not convert country code: ${countryCode}`, error);
            return countryCode;
        }
    };

    const getCurrencyName = (currencyCode) => {
        return currencyList.find((c) => c.currency_code == currencyCode)?.currency_abbreviation;
    }

    const fetchProposalItems = async (proposalId) => {
        try {
            const response = await request({
                // api: getProposalItemsAPI,
                api: getItemsByProposalId,
                params: { id: proposalId }
            });

            if (response.status === 200 && response.data) {
                console.log("Items received:", response.data);
                // Handle both response structures - items directly or nested in data
                const items = response.data.items || response.data.data || [];

                // Map the response items to match the expected structure if needed
                const formattedItems = items.map(item => {
                    // Check if this is a mixed rate based on custom_fee_enable
                    const isMixedRate = item.custom_fee_enable === 1;
                    const calculationMode = item.calculation_mode || 'flat';

                    let flatComponent = 0;
                    let percentageComponent = 0;

                    if (isMixedRate) {
                        // For mixed rates, extract components based on storage pattern
                        const primaryValue = parseFloat(item.fee || 0);
                        let secondaryValue = 0;

                        // Parse secondary component from JSON
                        if (item.custom_proposed_fee) {
                            try {
                                const customFeeData = JSON.parse(item.custom_proposed_fee);
                                secondaryValue = parseFloat(customFeeData.fees || 0);
                                console.log("DEBUG List.jsx - Mixed rate detected:", {
                                    item_id: item.item_id,
                                    calculation_mode: calculationMode,
                                    primary_fee: primaryValue,
                                    custom_proposed_fee_json: customFeeData,
                                    secondary_fee: secondaryValue,
                                    fee_percentage_field: item.fee_percentage
                                });
                            } catch (e) {
                                console.log("Failed to parse custom fee JSON in List.jsx", e);
                            }
                        }

                        // Map components based on calculation_mode
                        if (calculationMode === 'percentage') {
                            percentageComponent = primaryValue;    // Primary
                            flatComponent = secondaryValue;        // Secondary
                        } else {
                            flatComponent = primaryValue;          // Primary  
                            // For percentage component, prioritize fee_percentage field if it exists and is different from JSON
                            if (item.fee_percentage && parseFloat(item.fee_percentage) !== secondaryValue) {
                                console.log("DEBUG List.jsx - Using fee_percentage field instead of JSON:", {
                                    json_value: secondaryValue,
                                    fee_percentage_value: parseFloat(item.fee_percentage)
                                });
                                percentageComponent = parseFloat(item.fee_percentage);
                            } else {
                                percentageComponent = secondaryValue;  // Secondary from JSON
                            }
                        }
                    } else {
                        // Single rate handling - detect based on existing data
                        // If both fee and fee_percentage exist, it might be stored as separate fields
                        if (item.fee && item.fee_percentage) {
                            // Check if we have both values and neither is 0, this might indicate mixed rate stored differently
                            const feeValue = parseFloat(item.fee || 0);
                            const percentValue = parseFloat(item.fee_percentage || 0);

                            if (feeValue > 0 && percentValue > 0) {
                                // This looks like a mixed rate stored in separate fields
                                flatComponent = feeValue;
                                percentageComponent = percentValue;
                            } else if (calculationMode === 'percentage' || percentValue > 0) {
                                percentageComponent = percentValue || feeValue;
                            } else {
                                flatComponent = feeValue;
                            }
                        } else if (calculationMode === 'percentage' || item.fee_percentage) {
                            percentageComponent = parseFloat(item.fee_percentage || item.fee || 0);
                        } else {
                            flatComponent = parseFloat(item.fee || 0);
                        }
                    }

                    return {
                        ...item,
                        // Add any necessary transformations or defaults
                        item_id: item.item_id,
                        country_code: item.country_code,
                        payout_type: item.payout_type || item.payout_method || '',
                        fee_type: item.fee_type || '',
                        fee_flat: flatComponent,
                        fee_percentage: percentageComponent,
                        volume_type: item.volume_type || '',
                        committed_volume: item.committed_volume || item.commited_volume || item.comitted_volume || 0,
                        slab_from: item.min_value || 0,
                        slab_to: item.max_value >= 999999999 ? "Infinity" : item.max_value ?? "Infinity",
                        // Add mixed rate identification fields
                        custom_fee_enable: item.custom_fee_enable || 0,
                        calculation_mode: calculationMode,
                        is_mixed_rate: isMixedRate || (parseFloat(item.fee || 0) > 0 && parseFloat(item.fee_percentage || 0) > 0)
                    };
                });

                setLoadedItems(prev => ({
                    ...prev,
                    [proposalId]: formattedItems
                }));
            }
        } catch (error) {
            console.error(`Error fetching items for proposal ${proposalId}:`, error);
        }
    };

    // New function that returns items directly for PDF generation
    const fetchProposalItemsForPDF = async (proposalId) => {
        try {
            const response = await request({
                api: getItemsByProposalId,
                params: { id: proposalId }
            });

            if (response.status === 200 && response.data) {
                const items = response.data.items || response.data.data || [];

                const formattedItems = items.map(item => {
                    const isMixedRate = item.custom_fee_enable === 1;
                    const calculationMode = item.calculation_mode || 'flat';

                    let flatComponent = 0;
                    let percentageComponent = 0;

                    if (isMixedRate) {
                        const primaryValue = parseFloat(item.fee || 0);
                        let secondaryValue = 0;

                        if (item.custom_proposed_fee) {
                            try {
                                const customFeeData = JSON.parse(item.custom_proposed_fee);
                                secondaryValue = parseFloat(customFeeData.fees || 0);
                            } catch (e) {
                                console.log("Failed to parse custom fee JSON for PDF", e);
                            }
                        }

                        if (calculationMode === 'percentage') {
                            percentageComponent = primaryValue;
                            flatComponent = secondaryValue;
                        } else {
                            flatComponent = primaryValue;
                            if (item.fee_percentage && parseFloat(item.fee_percentage) !== secondaryValue) {
                                percentageComponent = parseFloat(item.fee_percentage);
                            } else {
                                percentageComponent = secondaryValue;
                            }
                        }
                    } else {
                        if (item.fee && item.fee_percentage) {
                            const feeValue = parseFloat(item.fee || 0);
                            const percentValue = parseFloat(item.fee_percentage || 0);

                            if (feeValue > 0 && percentValue > 0) {
                                flatComponent = feeValue;
                                percentageComponent = percentValue;
                            } else if (calculationMode === 'percentage' || percentValue > 0) {
                                percentageComponent = percentValue || feeValue;
                            } else {
                                flatComponent = feeValue;
                            }
                        } else if (calculationMode === 'percentage' || item.fee_percentage) {
                            percentageComponent = parseFloat(item.fee_percentage || item.fee || 0);
                        } else {
                            flatComponent = parseFloat(item.fee || 0);
                        }
                    }

                    return {
                        ...item,
                        item_id: item.item_id,
                        country_code: item.country_code,
                        payout_type: item.payout_type || item.payout_method || '',
                        fee_type: item.fee_type || '',
                        fee_flat: flatComponent,
                        fee_percentage: percentageComponent,
                        velocity_type: item.velocity_type || '',
                        committed_volume: item.committed_volume || item.commited_volume || item.comitted_volume || 0,
                        slab_from: item.slab_from || item.min_value || 0,
                        slab_to: item.slab_to || item.max_value || null,
                        custom_fee_enable: item.custom_fee_enable || 0,
                        calculation_mode: calculationMode,
                        is_mixed_rate: isMixedRate || (parseFloat(item.fee || 0) > 0 && parseFloat(item.fee_percentage || 0) > 0)
                    };
                });

                return formattedItems;
            }
            return [];
        } catch (error) {
            console.error(`Error fetching items for PDF ${proposalId}:`, error);
            throw error;
        }
    };

    const handleRowExpand = async (proposalId) => {
        setExpandedRows(prev => {
            const newSet = new Set(prev);
            if (newSet.has(proposalId)) {
                newSet.delete(proposalId);
            } else {
                newSet.add(proposalId);
                if (!loadedItems[proposalId]) {
                    fetchProposalItems(proposalId);
                }
            }
            return newSet;
        });
    };

    const handleEmailToPartner = async (proposalData) => {
        if (!proposalData) {
            setError("Proposal data not found");
            return;
        }

        // Get proposal items (fetch if not already loaded)
        let items = loadedItems[proposalData.id];
        if (!items) {
            items = await fetchProposalItemsForPDF(proposalData.id);
        }

        if (!items || items.length === 0) {
            setError("No proposal items found to include in document");
            return;
        }

        const partner = partnerList.find(
            (partner) => partner.partner_id == proposalData.partner_id
        );
        // Generate PDF with table styling
        const result = await generateProposalPDF(
            proposalData,
            items,
            partner.partner_name,
            paymentInstruments,
            false,
        );

        // setEmailPartnerDetails(partner);
        setEmailData(() => ({
            partner,
            subject: "Proposal - TerraPay",
            email: partner.email,
            proposal_id: proposalData.id,
            body: `<p>Dear ${partner.partner_name},<br><br>Please find the attached PDF containing the proposal from TerraPay.<br><br>Warm Regards,</p><p>TerraPay.</p>`,
            // body: `Dear ${partner.partner_name},<br/>Please find the attached PDF containing the proposal from TerraPay.<br/>Warm Regards,<br/>TerraPay.`,
            attachments: [{
                filename: result.fileName,
                content: result.base64?.replace(/^data:application\/pdf;base64,/, ''),
                encoding: 'base64'
            }]
        }));
        setEmailModal(true);
    };

    const fetchProposals = async () => {
        try {
            console.log("Fetching proposals...");
            const response = await request({
                api: getAllProposalAPI,
            });
            if (response.status == 200) {
                // Sort proposals by created_at date in descending order (newest first)
                const sortedProposals = response.data.data
                    .map((item, index) => ({
                        ...item,
                        srNo: index + 1,
                        status:  item.expiry_date && new Date(item.expiry_date) < new Date() ? "EXPIRED" : item.proposal_status
                    }))
                    .sort((a, b) => {
                        // Convert strings to Date objects and compare
                        return new Date(b.created_at) - new Date(a.created_at);
                    });

                setProposals(sortedProposals);
            }
            setLoading(false);
        } catch (error) {
            console.error("Error fetching proposals:", error);
            setError("Failed to fetch proposals");
            setLoading(false);
        }
    };

    const getStatusBgColor = (status) => {
        switch (status) {
            case "PENDING_APPROVAL":
                return alpha(Colors.tpPending, 0.1); // Light orange background
            case "APPROVED":
                return alpha(Colors.tpSuccess, 0.1); // Light green background
            case "REJECTED":
                return alpha(Colors.tpRejectColor, 0.1); // Light red background
            case "RECALLED":
                return alpha(Colors.tpLightGray, 0.1); // Light grey background
            case "EXPIRED":
                return alpha(Colors.tpLightGray, 0.1); // Light grey background
            case "SIGNED_OFF":
                return alpha(Colors.tpBlue, 0.1); // Light blue background
            default:
                return alpha(Colors.tpLightGray, 0.1);
        }
    };

    const getStatusTextColor = (status) => {
        switch (status) {
            case "PENDING_APPROVAL":
                return Colors.tpPending; // Orange
            case "APPROVED":
                return Colors.tpSuccess; // Green
            case "REJECTED":
                return Colors.tpRejectColor; // Red
            case "RECALLED":
                return Colors.tpTextLight; // Grey
            case "EXPIRED":
                return Colors.tpTextLight; // Grey
            case "SIGNED_OFF":
                return Colors.tpBlue; // Blue
            default:
                return Colors.tpTextLight;
        }
    };

    useEffect(() => {
        fetchProposals();
    }, []);

    // In Proposals.jsx, update the handleStatusUpdate function
    const handleStatusUpdate = async (comments) => {
        try {
            setError("");

            if (dialogState.type === "sign_off") {
                const response = await request({
                    api: updateProposalStatusAPI,
                    payload: {
                        proposal_id: dialogState.proposalId,
                        new_status: "SIGNED_OFF",
                        comments: comments || "", // Allow empty comments
                    },
                });
                // Reset dialog state and refresh proposals
                setDialogState({
                    open: false,
                    type: null,
                    proposalId: null,
                });
                fetchProposals();
                return;
            }

            // Rest of your existing status update logic
            if (dialogState.type === "move_to_pending") {
                const response = await request({
                    api: updateProposalStatusAPI,
                    payload: {
                        proposal_id: dialogState.proposalId,
                        comments: comments,
                        new_status: "PENDING_APPROVAL",
                    },
                });
            } else if (dialogState.type === "recall") {
                const response = await request({
                    api: updateProposalStatusAPI,
                    payload: {
                        proposal_id: dialogState.proposalId,
                        comments: comments,
                        new_status: "RECALLED",
                    },
                });
            } else {
                const status = dialogState.type === "approve" ? "APPROVED" : "REJECTED";
                const response = await request({
                    api: updateProposalStatusAPI,
                    payload: {
                        proposal_id: Number(dialogState.proposalId),
                        new_status: status,
                        comments: comments || "",
                    },
                });
            }

            setDialogState({
                open: false,
                type: null,
                proposalId: null,
            });
            fetchProposals();
        } catch (error) {
            console.error("Error updating proposal status:", error);
            setError(
                error.response?.data?.error || "Failed to update proposal status"
            );
        }
    };
    const handleDownloadPDF = async (proposalId) => {
        try {
            setError(""); // Clear any previous errors

            // Find the proposal data
            const proposalData = proposals.find(p => p.id === proposalId);
            if (!proposalData) {
                setError("Proposal data not found");
                return;
            }

            // Get proposal items (fetch if not already loaded)
            let items = loadedItems[proposalId];
            if (!items) {
                items = await fetchProposalItemsForPDF(proposalId);
            }

            if (!items || items.length === 0) {
                setError("No proposal items found to include in document");
                return;
            }

            // Get partner name
            const partnerName = partnerNames[proposalData.partner_id] || 'Unknown Partner';

            // Generate PDF with table styling
            const result = await generateProposalPDF(
                proposalData,
                items,
                partnerName,
                paymentInstruments
            );

            console.log("PDF generated successfully:", result.fileName);

        } catch (error) {
            console.error("Error generating document:", error);
            setError("Failed to generate document: " + (error.message || "Unknown error"));
        }
    };

    const handleDownloadExcel = async (proposalId) => {
        try {
            setError(""); // Clear any previous errors

            // Find the proposal data
            const proposalData = proposals.find(p => p.id === proposalId);
            if (!proposalData) {
                setError("Proposal data not found");
                return;
            }

            // Get proposal items (fetch if not already loaded)
            let items = loadedItems[proposalId];
            if (!items) {
                items = await fetchProposalItemsForPDF(proposalId);
            }

            if (!items || items.length === 0) {
                setError("No proposal items found to include in document");
                return;
            }

            // Get partner name
            const partnerName = partnerNames[proposalData.partner_id] || 'Unknown Partner';

            const excelData = items.map((item, index) => ({
              "#": index + 1,
              "Country": `${getCountryDisplayName(item.country_code)} (${item.country_code})`,
              "Payout Currency": item.currency || 'USD',
              "Transaction Type": transactionTypeOptions.find((option) => option.value === item.transaction_type)?.label || 'P2P',
              "Instrument": getPayoutTypeName(item.payout_type || item.payout_method),
              "Coverage": item.coverage || "All",
              "Rate (USD)": item.is_mixed_rate || item.custom_fee_enable === 1
                          ? `${Number(item.fee_flat || 0).toFixed(2)} USD + ${Number(item.fee_percentage || 0).toFixed(2)}%`
                          : item.fee_type === 'PERCENTAGE' || item.calculation_mode === 'percentage'
                              ? `${Number(item.fee_percentage || 0).toFixed(2)}%`
                              : `${Number(item.fee_flat || 0).toFixed(2)} USD`,
              "Velocity Type": getVelocityTypeName(item.velocity_type ?? 2),
              "Velocity Period": getVelocityPeriodName(item.velocity_period ?? 2),
              "Monthly Volume Commitment": item.committed_volume,
              "Volume Slab": getVolumeSlabName(item.min_value, item.max_value),
            }));

            CommonExcelDownload(partnerName + Date.now(), excelData);
        } catch (e) {
            console.error("Error generating excel document:", e);
        }
    }

    const sendEmail = async () => {
        dispatch(addLoader("SEND_EMAIL"));
        setEmailData((prev) => ({ ...prev, isLoading: true }));
        try {
            const response = await request({
                api: sendCustomEmailAPI,
                payload: {
                    to: emailData.email,
                    cc: emailData.cc,
                    bcc: emailData.bcc,
                    subject: emailData.subject,
                    message: emailData.body,
                    attachments: emailData.attachments,
                    proposal_id: emailData.proposal_id,
                }
            });
            if (response.status === 200 || response.status === 201) {
                setEmailData(() => (
                    { isLoading: false, email: "", subject: "", body: "", cc: "", bcc: "", attachments: [] }
                ));
                setEmailModal(false);
                renderToaster({
                    type: "success",
                    message: "Email sent successfully"
                });
            } else {
                setEmailData((prev) => ({ ...prev, isLoading: false }));
                renderToaster({
                    type: "error",
                    message: response.data?.message || "Failed to send email"
                });
            }
        } catch (e) {
            console.log("Error while sending email:", e);
        } finally {
            dispatch(removeLoader("SEND_EMAIL"));
            fetchProposals();
        }
    }

    const StatusUpdateDialog = () => {
        return (
            <CustomModal
                isOpen={dialogState.open}
                onClose={() =>
                    setDialogState({
                        open: false,
                        type: null,
                        proposalId: null,
                    })
                }
                onSubmit={handleStatusUpdate}
                title={
                    dialogState.type === "approve"
                        ? "Approve Proposal"
                        : dialogState.type === "reject"
                            ? "Reject Proposal"
                            : dialogState.type === "recall"
                                ? "Recall Proposal"
                                : dialogState.type === "move_to_pending"
                                    ? "Move to Pending Approval"
                                    : dialogState.type === "sign_off"
                                        ? "Sign Off Proposal"
                                        : "Update Proposal"
                }
                actionButtonText={
                    dialogState.type === "approve"
                        ? "Approve"
                        : dialogState.type === "reject"
                            ? "Reject"
                            : dialogState.type === "recall"
                                ? "Recall"
                                : dialogState.type === "move_to_pending"
                                    ? "Move to Pending"
                                    : dialogState.type === "sign_off"
                                        ? "Sign Off"
                                        : "Update"
                }
                isApprove={dialogState.type === "approve"}
                proposalId={dialogState.proposalId}
            />
        );
    };

    const handleRenegotiateClick = (proposalId, itemId) => {
        navigate(`/renegotiate-proposal/${proposalId}/${itemId}`);
    };

    if (loading) {
        return (
            <Box sx={{ p: 3 }}>
                <Typography>Loading proposals...</Typography>
            </Box>
        );
    }

    const columns = [
        {
            headerName: "",
            field: "expand",
            width: "48px",
            maxWidth: "48px",
            align: "center",
            renderCell: ({ row }) => (
                <IconButton
                    size="small"
                    onClick={(e) => {
                        e.stopPropagation();
                        handleRowExpand(row.id);
                    }}
                >
                    {expandedRows.has(row.id) ? <KeyboardArrowUpIcon /> : <KeyboardArrowDownIcon />}
                </IconButton>
            ),
        },
        {
            headerName: "Proposal ID",
            field: "proposal_name",
            width: "280px",
            maxWidth: "280px",
        },
        {
            headerName: "Partner Name",
            field: "partner_id",
            width: "230px",
            maxWidth: "230px",
            renderCell: ({ row }) => {
                return partnerNames[row.partner_id];
            },
        },
        {
            headerName: "Status",
            field: "proposal_status",
            align: "center",
            width: "200px",
            maxWidth: "200px",
            renderCell: ({ row }) => {
                return (
                    <Box
                        sx={{
                            display: "inline-flex",
                            alignItems: "center",
                            justifyContent: "center",
                            p: 0.5,
                            borderRadius: "4px",
                            // backgroundColor: getStatusBgColor(displayStatus),
                            color: getStatusTextColor(row?.status),
                            minWidth: "100%",
                            fontWeight: 500,
                            fontSize: "0.875rem",
                            fontFamily: "Space Grotesk",
                            textAlign: "center",
                        }}
                    >
                        {row?.status}
                    </Box>
                );
            },
        },
        {
            headerName: "Created at",
            field: "created_at",
            width: "200px",
            maxWidth: "200px",
            renderCell: ({ row }) => {
                return new Date(row.created_at)?.toLocaleString();
            },
        },
        {
            headerName: "Expiry",
            field: "expiry_date",
            width: "150px",
            maxWidth: "150px",
            renderCell: ({ row }) => {
                return new Date(row.expiry_date)?.toLocaleString();
            },
        },
        {
            headerName: "Version",
            field: "ver",
            width: "150px",
            maxWidth: "150px",
            align: "center",
            renderCell: ({ row }) => {
                return row.ver;
            },
        },
        {
            headerName: "Actions",
            field: "actions",
            width: "150px",
            align: "center",
            renderCell: ({ row }) => {
                return (
                    <Stack
                        direction="row"
                        gap={1}
                        ml={1}
                        // justifyContent={"center"}
                        sx={{ cursor: "pointer" }}
                    >
                        <Box
                            title="View"
                            onClick={(e) => {
                                e.preventDefault();
                                navigate(`${row.id}`);
                            }}
                            sx={{
                                cursor: "pointer",
                            }}
                        >
                            <i className="ri-eye-line" />
                        </Box>
                        {(row.proposal_status === "RECALLED" || row.proposal_status === "DRAFT") && (
                            <Box
                                title="Edit"
                                onClick={(e) => {
                                    e.preventDefault();
                                    navigate(`${row.id}`, { state: { edit: true } });
                                }}
                                sx={{
                                    cursor: "pointer",
                                }}
                            >
                                <i className="ri-file-edit-line" />
                            </Box>
                        )}
                        {row.proposal_status === "PENDING_APPROVAL" && (
                            <Box
                                title="Re-call"
                                onClick={(e) => {
                                    e.preventDefault();
                                    // navigate(`${row.id}`);
                                    setDialogState({
                                        open: true,
                                        type: "recall",
                                        proposalId: row.id,
                                        proposal: row,
                                    });
                                }}
                                sx={{
                                    cursor: "pointer",
                                }}
                            >
                                <i className="ri-restart-line" />
                            </Box>
                        )}
                        {!["PENDING_APPROVAL","RECALLED"].includes(row.proposal_status) && (
                            <Box
                                title="Download Proposal PDF"
                                onClick={(e) => {
                                    e.preventDefault();
                                    handleDownloadPDF(row.id);
                                }}
                                sx={{
                                    cursor: "pointer",
                                }}
                            >
                                <i className="ri-file-download-line" />
                            </Box>
                        )}
                        {!["PENDING_APPROVAL","RECALLED"].includes(row.proposal_status) && (
                            <Box
                                title="Download Proposal Excel"
                                onClick={(e) => {
                                    e.preventDefault();
                                    handleDownloadExcel(row.id);
                                }}
                            >
                                <i className="ri-download-line"></i>
                            </Box>
                        )}
                        <Box
                            title="Show History"
                            onClick={(e) => {
                                e.preventDefault();
                                navigate(`history/${row.id}`);
                                // navigate(`${row.id}`);
                            }}
                            sx={{
                                cursor: "pointer",
                            }}
                        >
                            <i className="ri-history-line" />
                        </Box>
                        {row.proposal_status === "APPROVED" && (
                        <Box
                            title="Send Mail"
                            onClick={(e) => {
                                e.preventDefault();
                                handleEmailToPartner(row);
                                // navigate(`history/${row.id}`);
                                // navigate(`${row.id}`);
                            }}
                            sx={{
                                cursor: "pointer",
                            }}
                        >
                            <i className="ri-mail-send-line" />
                        </Box>)
                        }
                    </Stack>
                );
            },
        },
    ];

    return (
        <Box sx={{ px: 3 }}>
            <Typography variant="h4" gutterBottom>
                Proposals
            </Typography>

            {error && (
                <Alert severity="error" sx={{ mb: 3 }}>
                    {error}
                </Alert>
            )}

            <Card variant="outlined" sx={{ mb: 3 }}>
                <CardHeader
                    title={
                        <Box
                            sx={{
                                display: "flex",
                                justifyContent: "space-between",
                                alignItems: "center",
                                width: "100%",
                            }}
                        >
                            <Typography variant="h6" sx={{ color: Colors.light }}>
                                All Proposals
                            </Typography>
                        </Box>
                    }
                    sx={{
                        bgcolor: Colors.tpBlue,
                        "& .MuiCardHeader-content": {
                            width: "100%",
                        },
                        p: 1.5,
                    }}
                />

                <FilterHeader
                    limit={limit}
                    setLimit={(newLimit) => {
                        setLimit(newLimit);
                        setPage(1); // Reset to page 1 when changing limit

                        // Update URL with new limit
                        try {
                            const params = new URLSearchParams(window.location.search);
                            params.set('limit', newLimit);
                            params.set('page', 1); // Reset page to 1

                            const newUrl = `${window.location.pathname}?${params.toString()}`;
                            window.history.replaceState({}, '', newUrl);
                        } catch (error) {
                            console.error("Error updating URL:", error);
                        }
                    }}
                    filters={[
                        {
                            name: "partner_id",
                            label: "Partner",
                            options: Object.entries(partnerNames).map(([key, value]) => ({
                                label: value,
                                value: key,
                            })),
                        },
                        {
                            name: "status",
                            label: "Status",
                            options: PROPOSAL_STATUS_OPTIONS,
                        },
                    ]}
                    addButton={
                        access.CREATE && (
                            <Button
                                variant="contained"
                                onClick={() => {
                                    navigate("/proposal-mgmt/proposals/add");
                                }}
                            >
                                Add Proposal
                            </Button>
                        )
                    }
                />
                <TableContainer>
                    <Table>
                        <TableHead>
                            <TableRow>
                                {columns.map((column) => (
                                    <TableCell
                                        key={column.field}
                                        align={column.align || 'left'}
                                        width={column.width}
                                        sx={{ maxWidth: column.maxWidth }}
                                    >
                                        {column.headerName}
                                    </TableCell>
                                ))}
                            </TableRow>
                        </TableHead>
                        <TableBody>
                            {/* {filteredProposals.slice(0, limit).map((row) => ( */}
                            {paginatedProposals.map((row) => (
                                <React.Fragment key={row.id}>
                                    <TableRow hover>
                                        {columns.map((column) => (
                                            <TableCell
                                                key={column.field}
                                                align={column.align || 'left'}
                                            >
                                                {column.renderCell
                                                    ? column.renderCell({ row })
                                                    : row[column.field]}
                                            </TableCell>
                                        ))}
                                    </TableRow>
                                    <TableRow>
                                        <TableCell style={{ paddingBottom: 0, paddingTop: 0 }} colSpan={columns.length}>
                                            <Collapse in={expandedRows.has(row.id)} timeout="auto" unmountOnExit>
                                                <Box sx={{
                                                    margin: 1,
                                                    maxHeight: '300px',
                                                    overflow: 'auto'
                                                }}>
                                                    <Typography variant="h6" gutterBottom component="div" sx={{ color: Colors.tpBlue }}>
                                                        Proposal Items
                                                    </Typography>
                                                    <TableContainer component={Paper}>
                                                        <Table size="small">
                                                            <TableHead sx={{ '& .MuiTableCell-root': { color: Colors.black } }}>
                                                                <TableRow>
                                                                    <TableCell align='center'>Item ID</TableCell>
                                                                    <TableCell align='center'>Country</TableCell>
                                                                    <TableCell align='center'>Payout Type</TableCell>
                                                                    <TableCell align='center'>Rate (USD)</TableCell>
                                                                    <TableCell align='center'>Velocity Type</TableCell>
                                                                    <TableCell align='center'>Velocity Period</TableCell>
                                                                    <TableCell align='center'>Monthly Volume<br />Commitment</TableCell>
                                                                    <TableCell align='center'>Volume Slab</TableCell>
                                                                    <TableCell align='center'>Actions</TableCell>
                                                                </TableRow>
                                                            </TableHead>
                                                            <TableBody>
                                                                {loadedItems[row.id] ? (
                                                                    loadedItems[row.id].map((item) => (
                                                                        <TableRow key={item.item_id}>
                                                                            <TableCell align='center'>{item.item_id}</TableCell>
                                                                            <TableCell align='center'>{`${getCountryDisplayName(item.country_code)} (${item.country_code})`}</TableCell>
                                                                            {/* <TableCell align='center'>{titleCase_(titleCaseSp(item.payout_type || ''))}</TableCell> */}
                                                                            <TableCell align='center'>
                                                                                {getPayoutTypeName(item.payout_type || item.payout_method)}
                                                                            </TableCell>
                                                                            <TableCell align='center'>
                                                                                {item.is_mixed_rate || item.custom_fee_enable === 1
                                                                                    ? `${Number(item.fee_flat || 0).toFixed(2)} USD + ${Number(item.fee_percentage || 0).toFixed(2)}%`
                                                                                    : item.fee_type === 'PERCENTAGE' || item.calculation_mode === 'percentage'
                                                                                        ? `${Number(item.fee_percentage || 0).toFixed(2)}%`
                                                                                        : `${Number(item.fee_flat || 0).toFixed(2)} USD`
                                                                                }
                                                                            </TableCell>
                                                                            <TableCell align='center'>
                                                                                {getVelocityTypeName(item.velocity_type ?? 2)}
                                                                            </TableCell>
                                                                            <TableCell align='center'>
                                                                                {getVelocityPeriodName(item.velocity_period ?? 2)}
                                                                            </TableCell>
                                                                            <TableCell align='center'>{item.committed_volume || 'No commitment'}</TableCell>
                                                                            <TableCell align='center'>{item.slab_from} - {item.slab_to || 'Unlimited'}</TableCell>
                                                                            <TableCell align='center'>
                                                                                {row.proposal_status === 'APPROVED' && (
                                                                                    <Button
                                                                                        size="small"
                                                                                        variant="contained"
                                                                                        color="info"
                                                                                        startIcon={<RefreshIcon />}
                                                                                        // onClick={() => handleRenegotiateClick(row.id, item.item_id)}
                                                                                        onClick={(e) => {
                                                                                            e.stopPropagation(); // Prevent row expansion
                                                                                            navigate(`/proposal-mgmt/proposals/renegotiate-proposal/${row.id}/${item.item_id}`);
                                                                                        }}
                                                                                    >
                                                                                        Request New Rate
                                                                                    </Button>
                                                                                )}
                                                                            </TableCell>
                                                                        </TableRow>
                                                                    ))
                                                                ) : (
                                                                    <TableRow>
                                                                        <TableCell colSpan={8} align="center">
                                                                            <CircularProgress size={20} />
                                                                        </TableCell>
                                                                    </TableRow>
                                                                )}
                                                            </TableBody>
                                                        </Table>
                                                    </TableContainer>
                                                </Box>
                                            </Collapse>
                                        </TableCell>
                                    </TableRow>
                                </React.Fragment>
                            ))}
                        </TableBody>
                    </Table>
                </TableContainer>

                {/* Pagination Control */}
                <Box sx={{
                    display: 'flex',
                    justifyContent: 'space-between',
                    alignItems: 'center',
                    p: 2,
                    borderTop: '1px solid rgba(224, 224, 224, 1)'
                }}>
                    <Box>
                        <Typography variant="body2" color="text.secondary">
                            Showing {paginatedProposals.length > 0 ? (page - 1) * limit + 1 : 0} to {Math.min(page * limit, filteredProposals.length)} of {filteredProposals.length} entries
                        </Typography>
                    </Box>
                    <Pagination
                        count={totalPages}
                        page={page}
                        onChange={handlePageChange}
                        color="primary"
                        showFirstButton
                        showLastButton
                    />
                </Box>

            </Card>

            <StatusUpdateDialog />
            {/* <RevokeDialog /> */}

            {/* Send email proposal to partner modal */}
            <EmailModal
                open={emailModal}
                onClose={(open) => setEmailModal(open)}
                emailData={emailData}
                setEmailData={setEmailData}
                emailError={emailError}
                setEmailError={setEmailError}
                sendEmail={sendEmail}
            />
        </Box>
    );
}

// src/modules-(pms)/pages/proposals/RenegotiateProposal.jsx
import React, { useState, useEffect } from 'react';
import { useParams, useNavigate } from 'react-router-dom';
import { getName } from 'country-list';
import { getCountryName } from "@modules-(pms)/utils/countryUtils";
import {
    Box,
    Card,
    CardContent,
    Typography,
    TextField,
    Button,
    Alert,
    Dialog,
    DialogTitle,
    DialogContent,
    DialogActions,
    FormControl,
    FormControlLabel,
    Radio,
    RadioGroup,
    Select,
    MenuItem,
    InputLabel,
    Grid,
    CircularProgress
} from '@mui/material';

import { request } from "@utils/index";
import { Colors } from "@/theme/colors";

import { getRenegotiationDetails, submitRenegotiationRequest } from "@services-(pms)/proposals";

function titleCaseSp(str) {
    if (!str) return '';
    const strValue = String(str);
    return strValue
        .toLowerCase()
        .split(' ')
        .map(word => word.charAt(0).toUpperCase() + word.slice(1))
        .join(' ');
}

function titleCase_(str) {
    if (!str) return '';
    const strValue = String(str);
    return strValue
        .toLowerCase()
        .split('_')
        .map(word => word.charAt(0).toUpperCase() + word.slice(1))
        .join(' ');
}

export default function RenegotiateProposal() {
    const { proposalId, itemId } = useParams();
    const navigate = useNavigate();

    // State management
    const [currentRate, setCurrentRate] = useState(null);
    const [requestType, setRequestType] = useState('STANDARD');
    const [reason, setReason] = useState('');
    const [error, setError] = useState('');
    const [success, setSuccess] = useState('');
    const [loading, setLoading] = useState(true);
    const [itemDetails, setItemDetails] = useState(null);
    const [customRateType, setCustomRateType] = useState('FLAT');
    const [customFlatRate, setCustomFlatRate] = useState('');
    const [customPercentageRate, setCustomPercentageRate] = useState('');
    const [walkawayRate, setWalkawayRate] = useState(null);
    const [warning, setWarning] = useState('');
    const [openConfirmModal, setOpenConfirmModal] = useState(false);
    const [rateHistory, setRateHistory] = useState(null);
    const [negotiationInfo, setNegotiationInfo] = useState(null);
    const [isAtMinimum, setIsAtMinimum] = useState(false);
    const [partnerDetails, setPartnerDetails] = useState(null);

    useEffect(() => {
        fetchItemDetails();
    }, [proposalId, itemId]);
    
    // Re-check for below minimum rates whenever rate type changes
    useEffect(() => {
        if (customRateType && itemDetails) {
            checkBelowMinimumRate();
        }
    }, [customRateType]);

    const fetchItemDetails = async () => {
        try {
            if (!proposalId || !itemId) {
                console.error('Missing required parameters');
                setError('Missing required parameters');
                setLoading(false);
                return;
            }

            console.log('Fetching item details for renegotiation:', { proposalId, itemId });

            const response = await request({
                api: getRenegotiationDetails,
                params: { proposalId, itemId }
            });

            console.log('Full API Response:', response);

            // Check if response has data and the request was successful
            if (response && response.status === 200 && response.data) {
                const proposalItem = response.data.proposal_item || response.data.data?.proposal_item;
                const negotiationInfo = response.data.negotiationInfo || response.data.data?.negotiationInfo;
                const isAtMinimum = response.data.isAtMinimum || response.data.data?.isAtMinimum;

                if (proposalItem) {
                    setItemDetails(proposalItem);

                    console.log('Fetched item details: ', proposalItem);

                    // Set current rate based on fee_type
                    if (proposalItem.fee_type === 'MIXED') {
                        setCurrentRate(`${parseFloat(proposalItem.fee_flat || 0).toFixed(2)} + ${parseFloat(proposalItem.fee_percentage || 0).toFixed(2)}%`);
                    } else if (proposalItem.fee_type === 'PERCENTAGE') {
                        setCurrentRate(`${parseFloat(proposalItem.fee_percentage || 0).toFixed(2)}%`);
                    } else {
                        // Default to FLAT if fee_type is missing or is FLAT
                        setCurrentRate(`${parseFloat(proposalItem.fee_flat || proposalItem.fee || 0).toFixed(2)}`);
                    }

                    // Set walkaway rate if available
                    if (proposalItem.minimum_fee_type === 'FLAT' || proposalItem.fee_type === 'FLAT') {
                        setWalkawayRate(parseFloat(proposalItem.minimum_fee_flat || proposalItem.minimum_fee || 0));
                    } else if (proposalItem.minimum_fee_type === 'PERCENTAGE' || proposalItem.fee_type === 'PERCENTAGE') {
                        setWalkawayRate(parseFloat(proposalItem.minimum_fee_percentage || proposalItem.minimum_fee || 0));
                    }
                }

                if (negotiationInfo) setNegotiationInfo(negotiationInfo);
                if (isAtMinimum !== undefined) setIsAtMinimum(isAtMinimum);
            } else {
                setError(response?.data?.message || 'Failed to fetch item details');
            }
            setLoading(false);
        } catch (error) {
            console.error('Error fetching item details:', error);
            setError('Failed to fetch item details: ' + (error.response?.data?.message || error.message));
            setLoading(false);
        }
    };

    const handleRateChange = (type, value) => {
        // Update the appropriate rate value
        if (type === 'flat') {
            setCustomFlatRate(value);
        } else {
            setCustomPercentageRate(value);
        }

        // Check for below minimum rate conditions based on rate type
        setTimeout(() => checkBelowMinimumRate(), 0);
    }

    // Function to check if rates are below minimum and set appropriate warnings
    const checkBelowMinimumRate = () => {
        if (!walkawayRate) return;
        
        // For FLAT rate type
        if (customRateType === 'FLAT') {
            const numericValue = parseFloat(customFlatRate || 0);
            if (numericValue < walkawayRate) {
                setWarning(`The entered value ${numericValue.toFixed(2)} is lower than the minimum/walkaway rate of ${walkawayRate.toFixed(2)}. This will require approval before being applied.`);
            } else {
                setWarning('');
            }
        }
        // For PERCENTAGE rate type
        else if (customRateType === 'PERCENTAGE') {
            const numericValue = parseFloat(customPercentageRate || 0);
            if (numericValue < walkawayRate) {
                setWarning(`The entered value ${numericValue.toFixed(2)}% is lower than the minimum/walkaway rate of ${walkawayRate.toFixed(2)}%. This will require approval before being applied.`);
            } else {
                setWarning('');
            }
        }
        // For MIXED rate type - need to extract the appropriate minimum values
        else if (customRateType === 'MIXED') {
            const flatValue = parseFloat(customFlatRate || 0);
            const percentageValue = parseFloat(customPercentageRate || 0);
            
            // We need to determine if either component is below minimum
            // This requires parsing the minimum values from itemDetails
            const minFlatRate = parseFloat(itemDetails?.minimum_fee_flat || itemDetails?.minimum_fee || 0);
            const minPercentageRate = parseFloat(itemDetails?.minimum_fee_percentage || itemDetails?.minimum_fee || 0);
            
            if (flatValue < minFlatRate || percentageValue < minPercentageRate) {
                setWarning(`One or more rate components are below the minimum/walkaway rate. This will require approval before being applied.`);
            } else {
                setWarning('');
            }
        }
    };

    const submitRequest = async () => {
        try {
            // Basic request data
            const requestData = {
                proposal_id: Number(proposalId),
                item_id: Number(itemId),
                request_type: requestType,
                reason: reason || ''
            };

            // Add custom rate data if applicable
            if (requestType === 'CUSTOM') {
                requestData.custom_rate_type = customRateType;

                switch (customRateType) {
                    case 'FLAT':
                        requestData.custom_flat_rate = parseFloat(customFlatRate).toFixed(2);
                        requestData.custom_percentage_rate = 0;
                        break;
                    case 'PERCENTAGE':
                        requestData.custom_flat_rate = 0;
                        requestData.custom_percentage_rate = parseFloat(customPercentageRate).toFixed(2);
                        break;
                    case 'MIXED':
                        requestData.custom_flat_rate = parseFloat(customFlatRate).toFixed(2);
                        requestData.custom_percentage_rate = parseFloat(customPercentageRate).toFixed(2);
                        break;
                }
            }

            console.log('Submitting request data:', requestData);

            const response = await request({
                api: submitRenegotiationRequest,
                payload: requestData
            });

            console.log('Server response:', response);

            if (response && response.status === 200) {
                // Show appropriate success message based on whether the rate was below minimum
                if (response.data.below_minimum_rate) {
                    setSuccess('Renegotiation request submitted and set to PENDING_APPROVAL. It will need to be approved by a pricing manager.');
                } else {
                    setSuccess('Renegotiation request submitted successfully');
                }
                setOpenConfirmModal(false);
                setTimeout(() => navigate(-1), 3000); // Give users a bit more time to read the status message
            } else {
                throw new Error(response?.data?.message || 'Failed to submit request');
            }

        } catch (error) {
            console.error('Error submitting request:', error);

            const errorMessage = error.response?.data?.message ||
                error.response?.data?.error ||
                error.message ||
                'Failed to submit request';

            setError(errorMessage);
            setOpenConfirmModal(false);
        }
    };

    const handleSubmitWithWarning = async () => {
        try {
            const existingRate = itemDetails?.fee_type === 'MIXED'
                ? `${Number(itemDetails.fee_flat || 0).toFixed(2)} + ${Number(itemDetails.fee_percentage || 0).toFixed(2)}%`
                : itemDetails?.fee_type === 'PERCENTAGE'
                    ? `${Number(itemDetails.fee_percentage || 0).toFixed(2)}%`
                    : `${Number(itemDetails.fee_flat || 0).toFixed(2)}`;

            const proposedRate = customRateType === 'FLAT'
                ? Number(customFlatRate).toFixed(2)
                : customRateType === 'PERCENTAGE'
                    ? `${Number(customPercentageRate).toFixed(2)}%`
                    : `${Number(customFlatRate).toFixed(2)} + ${Number(customPercentageRate).toFixed(2)}%`;

            setRateHistory({
                previousRate: existingRate,
                proposedRate: proposedRate,
                minimumRate: walkawayRate ? walkawayRate.toFixed(2) : '0.00'
            });

            // Show confirmation dialog with appropriate message
            setOpenConfirmModal(true);
            
            // Make sure the warning message reflects the pending approval requirement
            if (warning && !warning.includes('will require approval')) {
                setWarning(warning + ' This will require approval before being applied.');
            }
        } catch (error) {
            console.error('Error getting rate history:', error);
            setError('Failed to get rate history');
        }
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            if (warning) {
                await handleSubmitWithWarning();
            } else {
                await submitRequest();
            }
        } catch (error) {
            setError(error.response?.data?.message || error.response?.data?.error || error.message);
        }
    };

    const getCountryDisplayName = (countryCode) => {
        try {
            // Use utility function for consistent SEPA handling
            const name = getCountryName(countryCode);
            if (!name) {
                switch (countryCode) {
                    case 'AN': return 'Netherlands Antilles';
                    case 'XK': return 'Kosovo';
                    case 'IV': return "Côte d'Ivoire";
                    default: return countryCode;
                }
            }
            return name;
        } catch (error) {
            console.warn(`Could not convert country code: ${countryCode}`, error);
            return countryCode;
        }
    };

    if (loading) {
        return (
            <Box sx={{ display: 'flex', justifyContent: 'center', p: 5 }}>
                <CircularProgress />
            </Box>
        );
    }

    return (
        <Box sx={{ p: 3 }}>
            <Typography variant="h4" gutterBottom>
                Request Rate Renegotiation
            </Typography>

            {partnerDetails && (
                <Card sx={{ mb: 3 }}>
                    <CardContent>
                        <Typography variant="h6" gutterBottom>
                            Partner: {partnerDetails.company_name || partnerDetails.partner_name}
                        </Typography>
                        <Typography variant="body1" color="textSecondary">
                            Proposal ID: {itemDetails?.proposal_name || ''}-{itemDetails?.proposal_id || ''}
                        </Typography>
                    </CardContent>
                </Card>
            )}

            {itemDetails && (
                <Card sx={{ mb: 3 }}>
                    <CardContent>
                        <Typography variant="h6" gutterBottom color={Colors.tpBlue}>
                            Item Details
                        </Typography>
                        <Grid container spacing={2}>
                            <Grid item xs={6} md={3}>
                                <Typography variant="subtitle2" color="textSecondary">
                                    Item ID
                                </Typography>
                                <Typography>
                                    {itemDetails.item_id || ''}
                                </Typography>
                            </Grid>
                            <Grid item xs={6} md={3}>
                                <Typography variant="subtitle2" color="textSecondary">
                                    Country
                                </Typography>
                                <Typography>
                                    {itemDetails.country_code ? `${getCountryDisplayName(itemDetails.country_code)} (${itemDetails.country_code})` : ''}
                                </Typography>
                            </Grid>
                            <Grid item xs={6} md={3}>
                                <Typography variant="subtitle2" color="textSecondary">
                                    Negotiation Level
                                </Typography>
                                <Typography>
                                    {(() => {
                                        // Ensure we're handling numeric values correctly
                                        const level = typeof itemDetails.negotiation_level === 'string'
                                            ? parseInt(itemDetails.negotiation_level, 10)
                                            : itemDetails.negotiation_level;

                                        switch (level) {
                                            case 0:
                                                return 'Rack Rate';
                                            case 1:
                                                return 'Negotiation Level 1';
                                            case 2:
                                                return 'Negotiation Level 2';
                                            case 3:
                                                return 'Walkaway Rate';
                                            case 4:
                                                return 'Custom Rate';
                                            default:
                                                return itemDetails.negotiation_level || '';
                                        }
                                    })()}
                                </Typography>
                            </Grid>
                            <Grid item xs={6} md={3}>
                                <Typography variant="subtitle2" color="textSecondary">
                                    Current Rate
                                </Typography>
                                <Typography>
                                    {(() => {
                                        const feeType = itemDetails?.fee_type;
                                        const feeFlat = parseFloat(itemDetails?.fee_flat || itemDetails?.fee || 0);
                                        const feePercentage = parseFloat(itemDetails?.fee_percentage || 0);

                                        if (feeType === 'MIXED') {
                                            return `${feeFlat.toFixed(2)} + ${feePercentage.toFixed(2)}%`;
                                        } else if (feeType === 'PERCENTAGE') {
                                            return `${feePercentage.toFixed(2)}%`;
                                        } else {
                                            return `${feeFlat.toFixed(2)}`;
                                        }
                                    })()} USD
                                </Typography>
                            </Grid>
                            <Grid item xs={6} md={3}>
                                <Typography variant="subtitle2" color="textSecondary">
                                    Payout Method
                                </Typography>
                                <Typography>
                                    {itemDetails.payout_type ? titleCaseSp(titleCase_(itemDetails.payout_type)) : ''}
                                </Typography>
                            </Grid>
                            <Grid item xs={6} md={3}>
                                <Typography variant="subtitle2" color="textSecondary">
                                    Fee Type
                                </Typography>
                                <Typography>
                                    {itemDetails?.fee_type ? titleCaseSp(titleCase_(itemDetails.fee_type)) : 'Flat'}
                                </Typography>
                            </Grid>

                            <Grid item xs={12}>
                                <Typography variant="h6" gutterBottom color={Colors.tpBlue} sx={{ mt: 2 }}>
                                    Volume Information
                                </Typography>
                                <Grid container spacing={2}>
                                    <Grid item xs={12} md={3}>
                                        <Typography variant="subtitle2" color="textSecondary">Volume Type</Typography>
                                        <Typography>
                                            {titleCaseSp(titleCase_(itemDetails?.volume_type || 'AMOUNT'))}
                                        </Typography>
                                    </Grid>
                                    <Grid item xs={12} md={3}>
                                        <Typography variant="subtitle2" color="textSecondary">Time Period</Typography>
                                        <Typography>
                                            {(() => {
                                                const timePeriod = itemDetails?.time_period;
                                                if (!timePeriod) return 'Month';

                                                switch (timePeriod) {
                                                    case 'MONTH': return 'Monthly';
                                                    case 'YEAR': return 'Yearly';
                                                    case 'DAY': return 'Daily';
                                                    case 'WEEK': return 'Weekly';
                                                    default: return titleCase_(timePeriod);
                                                }
                                            })()}
                                        </Typography>
                                    </Grid>
                                    <Grid item xs={12} md={3}>
                                        <Typography variant="subtitle2" color="textSecondary">Volume Slab</Typography>
                                        <Typography>
                                            {itemDetails?.slab_from || itemDetails?.min_value || '0'}{" - "}
                                            {itemDetails?.slab_to ? itemDetails.slab_to.toLocaleString() :
                                                itemDetails?.max_value ? itemDetails.max_value.toLocaleString() : 'Unlimited'}
                                        </Typography>
                                    </Grid>
                                    {itemDetails?.committed_volume && (
                                        <Grid item xs={12} md={3}>
                                            <Typography variant="subtitle2" color="textSecondary">Volume Commitment</Typography>
                                            <Typography>
                                                {itemDetails.committed_volume.toLocaleString()}
                                            </Typography>
                                        </Grid>
                                    )}
                                </Grid>
                            </Grid>
                        </Grid>
                    </CardContent>
                </Card>
            )}

            {error && (
                <Alert severity="error" sx={{ mb: 2 }}>
                    {error}
                </Alert>
            )}

            {warning && (
                <Alert severity="warning" sx={{ mb: 2 }}>
                    {warning}
                </Alert>
            )}

            {success && (
                <Alert severity="success" sx={{ mb: 2 }}>
                    {success}
                </Alert>
            )}

            <Card>
                <CardContent>
                    <form onSubmit={handleSubmit}>
                        <FormControl component="fieldset" sx={{ mb: 3, width: '100%' }}>
                            <Typography variant="h6" gutterBottom color={Colors.tpBlue}>
                                Request Type
                            </Typography>

                            <RadioGroup
                                value={requestType}
                                onChange={(e) => setRequestType(e.target.value)}
                            >
                                <FormControlLabel
                                    value="STANDARD"
                                    control={<Radio />}
                                    label="Standard Rate Negotiation"
                                    disabled={isAtMinimum}
                                />
                                <FormControlLabel
                                    value="CUSTOM"
                                    control={<Radio />}
                                    label="Custom Rate"
                                />
                            </RadioGroup>

                            {isAtMinimum && (
                                <Alert severity="warning" sx={{ mt: 1 }}>
                                    This item is at custom or minimum rate, please use custom rate for renegotiation
                                    as no standard rates are available for negotiation.
                                </Alert>
                            )}

                            {requestType === 'STANDARD' && !isAtMinimum && negotiationInfo && (
                                <Box sx={{ mt: 2, p: 2, bgcolor: 'background.paper', borderRadius: 1, border: `1px solid ${Colors.tpBorderColor}` }}>
                                    <Typography variant="subtitle2" gutterBottom>
                                        Rate details:
                                    </Typography>
                                    <Typography>
                                        Current Rate: USD 
                                            {(() => {
                                                const feeType = itemDetails?.fee_type;
                                                const feeFlat = parseFloat(itemDetails?.fee_flat || itemDetails?.fee || 0);
                                                const feePercentage = parseFloat(itemDetails?.fee_percentage || 0);

                                                if (feeType === 'MIXED') {
                                                    return `${feeFlat.toFixed(2)} + ${feePercentage.toFixed(2)}%`;
                                                } else if (feeType === 'PERCENTAGE') {
                                                    return `${feePercentage.toFixed(2)}%`;
                                                } else {
                                                    return `${feeFlat.toFixed(2)}`;
                                                }
                                            })()}
                                    </Typography>
                                    <Typography>
                                        Current Level: {
                                            (() => {
                                                const level = typeof itemDetails?.negotiation_level === 'string'
                                                    ? parseInt(itemDetails.negotiation_level, 10)
                                                    : itemDetails?.negotiation_level;

                                                switch (level) {
                                                    case 0: return 'Rack Rate';
                                                    case 1: return 'Negotiation Level 1';
                                                    case 2: return 'Negotiation Level 2';
                                                    case 3: return 'Walkaway Rate';
                                                    case 4: return 'Custom Rate';
                                                    default: return '';
                                                }
                                            })()
                                        }
                                    </Typography>
                                    <Typography sx={{ mt: 2, color: 'primary.main' }}>
                                        Next Rate: USD {negotiationInfo?.rate || 'N/A'}
                                    </Typography>
                                    <Typography sx={{ color: 'primary.main' }}>
                                        Next Level: {negotiationInfo?.level === 'RACK' ? 'Rack Rate' :
                                            negotiationInfo?.level === '1' ? 'Negotiation Level 1' :
                                                negotiationInfo?.level === '2' ? 'Negotiation Level 2' :
                                                    'Walkaway Rate'}
                                    </Typography>
                                </Box>
                            )}
                        </FormControl>

                        {requestType === 'CUSTOM' && (
                            <Box sx={{ mb: 3 }}>
                                <FormControl fullWidth sx={{ mb: 2 }}>
                                    <InputLabel>Rate Type</InputLabel>
                                    <Select
                                        value={customRateType}
                                        onChange={(e) => setCustomRateType(e.target.value)}
                                        required
                                        label="Rate Type"
                                    >
                                        <MenuItem value="FLAT">Flat Fee</MenuItem>
                                        <MenuItem value="PERCENTAGE">Percentage</MenuItem>
                                        <MenuItem value="MIXED">Mixed</MenuItem>
                                    </Select>
                                </FormControl>

                                {(customRateType === 'FLAT' || customRateType === 'MIXED') && (
                                    <TextField
                                        fullWidth
                                        label="Flat Rate"
                                        type="number"
                                        value={customFlatRate}
                                        onChange={(e) => handleRateChange('flat', e.target.value)}
                                        sx={{ mb: 2 }}
                                        required
                                        inputProps={{
                                            step: "0.001",
                                            min: "0.001"
                                        }}
                                    />
                                )}

                                {(customRateType === 'PERCENTAGE' || customRateType === 'MIXED') && (
                                    <TextField
                                        fullWidth
                                        label="Percentage Rate"
                                        type="number"
                                        value={customPercentageRate}
                                        onChange={(e) => handleRateChange('percentage', e.target.value)}
                                        sx={{ mb: 2 }}
                                        required
                                        inputProps={{
                                            step: "0.01",
                                            min: "0"
                                        }}
                                    />
                                )}
                            </Box>
                        )}

                        <TextField
                            fullWidth
                            label="Justification for Request"
                            multiline
                            rows={4}
                            value={reason}
                            onChange={(e) => setReason(e.target.value)}
                            required
                            sx={{ mb: 3 }}
                            placeholder="Please provide a brief justification for the rate renegotiation request. This justification will be visible to the pricing manager."
                        />

                        <Box sx={{ display: 'flex', gap: 2 }}>
                            <Button
                                variant="outlined"
                                onClick={() => navigate(-1)}
                            >
                                Cancel
                            </Button>

                            <Button
                                type="submit"
                                variant="contained"
                                color="primary"
                                disabled={!reason || (requestType === 'CUSTOM' && (
                                    (customRateType === 'FLAT' && !customFlatRate) ||
                                    (customRateType === 'PERCENTAGE' && !customPercentageRate) ||
                                    (customRateType === 'MIXED' && (!customFlatRate || !customPercentageRate))
                                ))}
                            >
                                Submit Request
                            </Button>
                        </Box>
                    </form>
                </CardContent>
            </Card>

            {/* Confirmation Dialog */}
            <Dialog open={openConfirmModal} onClose={() => setOpenConfirmModal(false)}>
                <DialogTitle sx={{ bgcolor: Colors.tpBlue, color: Colors.light }}>
                    Confirm Rate Request
                </DialogTitle>
                <DialogContent>
                    {rateHistory && (
                        <Box sx={{ mt: 2 }}>
                            <Typography sx={{ fontWeight: 'bold', mb: 2 }}>
                                Requesting Rate: USD {rateHistory.proposedRate}
                            </Typography>
                            <Alert severity="warning" sx={{ mb: 2 }}>
                                Warning: The rate you are about to request is below the recommended minimum walkaway rate. 
                                This proposal will be set to "PENDING_APPROVAL" status and will require manager approval before taking effect.
                            </Alert>
                            <Typography variant="body2" sx={{ mb: 2 }}>
                                Are you sure you want to proceed with this request?
                            </Typography>
                        </Box>
                    )}
                </DialogContent>
                <DialogActions>
                    <Button onClick={() => setOpenConfirmModal(false)}>Cancel</Button>
                    <Button onClick={submitRequest} color="primary" variant="contained">
                        Submit Request for Approval
                    </Button>
                </DialogActions>
            </Dialog>
        </Box>
    );
}
import React, { useState, useEffect } from "react";
import { useParams, useNavigate } from "react-router-dom";
import {
  Box,
  Paper,
  Typography,
  Button,
  Grid,
  Alert,
  Card,
  CardContent,
  IconButton,
  TextField,
  CircularProgress,
  Dialog,
  DialogTitle,
  DialogContent,
  DialogActions,
  FormControl,
  InputLabel,
  Select,
  MenuItem,
} from "@mui/material";
import { Add as AddIcon, Delete as DeleteIcon } from "@mui/icons-material";
import { getName, getCode } from "country-list";
import { getCountryName } from "@modules-(pms)/utils/countryUtils";
import { request } from "@utils/index";
import {
  getProposalByID,
  getProposalHistoryByID,
  updateProposalStatusAPI,
} from "@services-(pms)/proposals";
// import { getName, getCode } from '../../../server/src/utils/countryCodeConverter';

function titleCase_(str = "") {
  return str
    .toLowerCase()
    .split("_")
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
    .join(" ");
}
function titleCaseSp(str) {
  return str
    .toLowerCase()
    .split(" ")
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
    .join(" ");
}

export default function ReviewProposal() {
  const { id } = useParams();
  const navigate = useNavigate();
  const [proposal, setProposal] = useState(null);
  const [rejectionReason, setRejectionReason] = useState("");
  const [error, setError] = useState("");
  const [success, setSuccess] = useState("");
  const [loading, setLoading] = useState(true);
  const [editingItem, setEditingItem] = useState(null);
  const user = JSON.parse(sessionStorage.getItem("user"));

  const [originalRates, setOriginalRates] = useState({});

  // New states for modal and form
  const [addItemModal, setAddItemModal] = useState(false);
  const [countries, setCountries] = useState([]);
  const [payoutMethods, setPayoutMethods] = useState([]);
  const [newItem, setNewItem] = useState({
    country_code: "",
    country_name: "",
    payout_method: "",
    proposed_rate: "",
    reason: "",
  });

  const [duplicateWarning, setDuplicateWarning] = useState("");

  useEffect(() => {
    if (id) {
      fetchProposalDetails();
      fetchCountriesAndMethods();
    }
  }, [id]);

  const fetchProposalDetails = async () => {
    try {
      const proposalRes = await request({
        api: getProposalByID,
        params: { id },
      });

      const historyRes = await request({
        api: getProposalHistoryByID,
        params: { id },
      });

      const origRates = {};
      proposalRes.data.items?.forEach((item) => {
        origRates[item.item_id] = item.proposed_rate;
      });
      setOriginalRates(origRates);

      setProposal({
        ...proposalRes.data.proposal,
        items: proposalRes.data.items || [],
      });

      const rejectionEntry = historyRes.data?.history?.find((h) => {
        if (h.change_type !== "STATUS_CHANGED") return false;
        let newData;
        try {
          newData =
            typeof h.new_data === "string"
              ? JSON.parse(h.new_data)
              : h.new_data;
        } catch (e) {
          return false;
        }
        return newData?.status === "REJECTED";
      });

      const rejectionReason =
        rejectionEntry?.change_reason || "No reason provided";

      setProposal({
        ...proposalRes.data.proposal,
        items: proposalRes.data.items || [],
      });

      setRejectionReason(rejectionReason);
      setLoading(false);
    } catch (error) {
      console.error("Error fetching proposal details:", error);
      setError("Failed to fetch proposal details. Please try again.");
      setLoading(false);
    }
  };

  // useEffect(() => {
  //     if (id) {
  //         fetchProposalDetails();
  //         fetchCountriesAndMethods();
  //     }
  // }, [id]);

  const fetchCountriesAndMethods = async () => {
    try {
      // const countriesResponse = await axiosInstance.get("/countries");
      // setCountries(countriesResponse.data.countries);
    } catch (error) {
      console.error("Error fetching countries:", error);
      setError("Failed to fetch countries. Please try again.");
    }
  };

  const handleEditRate = (item) => {
    setEditingItem({
      ...item,
      proposed_rate: item.proposed_rate || "",
    });
  };

  const handleCancelEdit = () => {
    setEditingItem(null);
    setError(null);
  };

  const handleRateUpdate = async (itemId, newRate) => {
    try {
      // Validate the new rate
      if (isNaN(newRate) || newRate < 0) {
        setError("Please enter a valid rate");
        return;
      }

      // Update the item locally first
      setProposal((prev) => ({
        ...prev,
        items: prev.items.map((item) =>
          item.item_id === itemId ? { ...item, proposed_rate: newRate } : item
        ),
      }));

      // Clear editing state
      setEditingItem(null);
      setError(null);
    } catch (error) {
      console.error("Error updating rate:", error);
      setError("Failed to update rate. Please try again.");
    }
  };

  const handleCountryChange = async (country) => {
    try {
      const countryCode = getCode(country);

      if (!countryCode) {
        console.error("Could not find country code for:", country);
        return;
      }

      // Clear duplicate warning when country changes
      setDuplicateWarning("");

      // const methodsResponse = await axiosInstance.get(
      //   `/countries/payout-types/${countryCode}`
      // );
      // setPayoutMethods(methodsResponse.data.payout_types || []);

      setNewItem((prev) => ({
        ...prev,
        country_code: countryCode,
        country_name: country, // This is important to keep the selected country visible
        payout_method: "", // Reset payout method when country changes
        proposed_rate: "",
      }));
    } catch (error) {
      console.error("Error fetching payout methods:", error);
      setError("Failed to fetch payout methods. Please try again.");
    }
  };

  const handlePayoutMethodChange = async (payoutMethod) => {
    try {
      // Check for duplicates first
      if (newItem.country_code && payoutMethod) {
        const isDuplicate = proposal.items.some(
          (item) =>
            item.country_code.toLowerCase() ===
              newItem.country_code.toLowerCase() &&
            item.payout_method.toLowerCase() === payoutMethod.toLowerCase()
        );

        if (isDuplicate) {
          setDuplicateWarning(
            `${getCountryName(newItem.country_code)} with ${titleCase_(
              payoutMethod
            )} payout method already exists in this proposal`
          );
        } else {
          setDuplicateWarning("");
        }
      }

      // Always update the payout method selection
      setNewItem((prev) => ({
        ...prev,
        payout_method: payoutMethod,
      }));

      // If not duplicate, fetch and set the proposed rate
      if (!duplicateWarning) {
        // const response = await axiosInstance.get("/pricing/calculate-pricing", {
        //   params: {
        //     country: newItem.country_code,
        //     payout_type: payoutMethod,
        //   },
        // });
        // setNewItem((prev) => ({
        //   ...prev,
        //   proposed_rate: response.data.fees.proposed.display,
        // }));
      }
    } catch (error) {
      console.error("Error fetching pricing:", error);
      setError("Failed to fetch pricing. Please try again.");
    }
  };
  const handleAddItem = async () => {
    try {
      // Validate required fields
      if (
        !newItem.country_code ||
        !newItem.payout_method ||
        !newItem.proposed_rate ||
        !newItem.reason
      ) {
        setError("Please fill in all required fields");
        return;
      }

      // Check for duplicate country and payout method combination
      const isDuplicate = proposal.items.some(
        (item) =>
          item.country_code.toLowerCase() ===
            newItem.country_code.toLowerCase() &&
          item.payout_method.toLowerCase() ===
            newItem.payout_method.toLowerCase()
      );

      if (isDuplicate) {
        setDuplicateWarning(
          `${getCountryName(newItem.country_code)} with ${titleCase_(
            newItem.payout_method
          )} payout method already exists in this proposal`
        );
        return;
      } else {
        setDuplicateWarning("");
      }

      const user = JSON.parse(sessionStorage.getItem("user"));

      // Add the item - using 'id' from useParams instead of proposalId
      // const addResponse = await axiosInstance.post(
      //   `/proposals/add-proposal-item/${id}`,
      //   {
      //     item: {
      //       country_code: newItem.country_code,
      //       payout_method: newItem.payout_method,
      //       proposed_rate: newItem.proposed_rate,
      //       payout_currency: "USD",
      //     },
      //     reason: newItem.reason,
      //     username: user.username,
      //   }
      // );

      // if (addResponse.data.message) {
      // Update proposal status to PENDING_APPROVAL
      await request({
        api: updateProposalStatusAPI,
        payload: {
          proposal_id: Number(id),
          new_status: "PENDING_APPROVAL",
          comments: `Item added: ${getCountryName(newItem.country_code)} - ${
            newItem.payout_method
          }`,
        },
      });

      setSuccess(
        "Item added successfully. Proposal status updated to Pending Approval"
      );

      // Close modal and reset form
      setAddItemModal(false);
      setNewItem({
        country_code: "",
        country_name: "",
        payout_method: "",
        proposed_rate: "",
        reason: "",
      });

      // Refresh the proposal data
      await fetchProposalDetails();

      // Navigate back to proposals page after a short delay
      setTimeout(() => {
        navigate(-1);
      }, 2000);
      // }
    } catch (error) {
      console.error("Error adding item:", error);
      setError(error.response?.data?.error || "Failed to add item");
    }
  };
  const handleRemoveItem = async (itemId) => {
    try {
      const reason = prompt("Please provide a reason for deletion:");
      if (!reason || !reason.trim()) {
        setError("Deletion canceled. Reason is required.");
        return;
      }

      const confirmDelete = window.confirm(
        "Are you sure you want to remove this item?"
      );
      if (!confirmDelete) return;

      const user = JSON.parse(sessionStorage.getItem("user"));
      // const response = await axiosInstance.delete(
      //   `/proposals/proposal-items/${itemId}`,
      //   {
      //     params: {
      //       reason,
      //       username: user.username,
      //       proposal_id: Number(id),
      //     },
      //   }
      // );

      // if (response.status === 200) {
      //   setSuccess("Item removed successfully.");
      //   await fetchProposalDetails();
      //   setTimeout(() => setSuccess(""), 3000);
      // }
    } catch (error) {
      console.error("Error removing item:", error);
      setError(
        error.response?.data?.error ||
          "Failed to remove item. Please try again."
      );
    }
  };

  const handleSubmit = async () => {
    try {
      setError(null);

      console.log("Submitting proposal:", proposal);

      // First update any modified items
      // if (proposal.items.length > 0) {
      //   for (const item of proposal.items) {
      //     console.log("Updating item:", item);
      //     await axiosInstance.put(
      //       `/proposals/update-proposal-item/${item.proposal_id}/${item.item_id}`,
      //       {
      //         current_rate: originalRates[item.item_id],
      //         proposed_rate: item.proposed_rate,
      //         item_version: item.item_version || 1.0,
      //         username: user.username,
      //       }
      //     );
      //   }
      // }

      // Then update proposal status back to PENDING_APPROVAL
      await request({
        api: updateProposalStatusAPI,
        payload: {
          proposal_id: id,
          new_status: "PENDING_APPROVAL",
          comments: "Proposal resubmitted after review with modifications",
        },
      });

      setSuccess("Proposal updated successfully");
      setTimeout(() => {
        navigate(-1);
      }, 1500);
    } catch (error) {
      console.error("Error updating proposal:", error);
      setError(
        error.response?.data?.error ||
          "Failed to update proposal. Please try again."
      );
    }
  };

  if (loading) {
    return (
      <Box
        sx={{
          display: "flex",
          justifyContent: "center",
          alignItems: "center",
          minHeight: "60vh",
        }}
      >
        <CircularProgress />
      </Box>
    );
  }

  if (error && !proposal) {
    return (
      <Box sx={{ p: 3 }}>
        <Alert
          severity="error"
          action={
            <Button
              color="inherit"
              size="small"
              onClick={() => navigate(-1)}
            >
              Return to Proposals
            </Button>
          }
        >
          {error}
        </Alert>
      </Box>
    );
  }

  return (
    <Box sx={{ p: 3 }}>
      <Typography variant="h4" gutterBottom>
        Review Rejected Proposal
      </Typography>

      {proposal && (
        <>
          <Card sx={{ mb: 3 }}>
            <CardContent>
              <Grid container spacing={2}>
                <Grid item xs={12} md={6}>
                  <Typography variant="subtitle2" color="textSecondary">
                    Partner ID:
                  </Typography>
                  <Typography variant="body1">{proposal.partner_id}</Typography>
                </Grid>
                <Grid item xs={12} md={6}>
                  <Typography variant="subtitle2" color="textSecondary">
                    Proposal Name:
                  </Typography>
                  <Typography variant="body1">
                    {proposal.proposal_name}
                  </Typography>
                </Grid>
              </Grid>
            </CardContent>
          </Card>

          <Card sx={{ mb: 3, bgcolor: "#FFF4F4" }}>
            <CardContent>
              <Typography variant="h6" color="error" gutterBottom>
                Reason for Rejection
              </Typography>
              <Box sx={{ mt: 1 }}>
                <Typography
                  variant="body1"
                  sx={{
                    whiteSpace: "pre-wrap",
                    wordBreak: "break-word",
                  }}
                >
                  {rejectionReason}
                </Typography>
                {!rejectionReason && (
                  <Typography color="text.secondary" variant="body2">
                    No rejection reason provided
                  </Typography>
                )}
              </Box>
            </CardContent>
          </Card>

          <Paper sx={{ p: 3, mb: 3, overflowX: "auto" }}>
            <Box
              sx={{
                display: "flex",
                justifyContent: "space-between",
                alignItems: "center",
                mb: 2,
              }}
            >
              <Typography variant="h6">Proposal Items</Typography>
              <Button
                startIcon={<AddIcon />}
                variant="contained"
                onClick={() => setAddItemModal(true)}
                color="primary"
              >
                Add Country
              </Button>
            </Box>

            <Box sx={{ mb: 2 }}>
              {error && (
                <Alert severity="error" sx={{ mb: 2 }}>
                  {error}
                </Alert>
              )}
              {success && (
                <Alert severity="success" sx={{ mb: 2 }}>
                  {success}
                </Alert>
              )}
            </Box>

            <Box sx={{ minWidth: 700 }}>
              {proposal.items && proposal.items.length > 0 ? (
                proposal.items.map((item) => (
                  <Box
                    key={item.item_id}
                    sx={{
                      mb: 2,
                      p: 2,
                      border: "1px solid #e0e0e0",
                      borderRadius: 1,
                      "&:hover": { bgcolor: "#f5f5f5" },
                    }}
                  >
                    <Grid container spacing={2} alignItems="center">
                      <Grid item xs={12} sm={3}>
                        <Typography variant="subtitle2" color="textSecondary">
                          Country:
                        </Typography>
                        <Typography>
                          {getCountryName(item.country_code) +
                            " (" +
                            item.country_code +
                            ")" || item.country_code}
                        </Typography>
                      </Grid>
                      <Grid item xs={12} sm={2}>
                        <Typography variant="subtitle2" color="textSecondary">
                          Payout Method:
                        </Typography>
                        <Typography>
                          {titleCaseSp(titleCase_(item.payout_method))}
                        </Typography>
                      </Grid>
                      <Grid item xs={12} sm={5}>
                        <Typography variant="subtitle2" color="textSecondary">
                          Rate (USD):
                        </Typography>
                        {editingItem?.item_id === item.item_id ? (
                          <Box
                            sx={{
                              display: "flex",
                              gap: 1,
                              alignItems: "center",
                            }}
                          >
                            <TextField
                              size="small"
                              value={editingItem.proposed_rate}
                              onChange={(e) =>
                                setEditingItem({
                                  ...editingItem,
                                  proposed_rate: e.target.value,
                                })
                              }
                              sx={{ width: "150px" }}
                            />
                            <Button
                              size="small"
                              onClick={() =>
                                handleRateUpdate(
                                  item.item_id,
                                  editingItem.proposed_rate
                                )
                              }
                              color="primary"
                              variant="contained"
                            >
                              Save
                            </Button>
                            <Button size="small" onClick={handleCancelEdit}>
                              Cancel
                            </Button>
                          </Box>
                        ) : (
                          <Box
                            sx={{
                              display: "flex",
                              gap: 1,
                              alignItems: "center",
                            }}
                          >
                            <Typography>{item.proposed_rate}</Typography>
                            <Button
                              size="small"
                              onClick={() => handleEditRate(item)}
                              color="primary"
                            >
                              Edit
                            </Button>
                          </Box>
                        )}
                      </Grid>
                      <Grid item xs={12} sm={2} sx={{ textAlign: "right" }}>
                        <IconButton
                          color="error"
                          onClick={() => handleRemoveItem(item.item_id)}
                          title="Remove Item"
                        >
                          <DeleteIcon />
                        </IconButton>
                      </Grid>
                    </Grid>
                  </Box>
                ))
              ) : (
                <Typography
                  color="textSecondary"
                  sx={{ textAlign: "center", py: 3 }}
                >
                  No proposal items found. Click "Add Country" to add items.
                </Typography>
              )}
            </Box>
          </Paper>

          {/* Add Item Modal */}
          <Dialog
            open={addItemModal}
            onClose={() => setAddItemModal(false)}
            maxWidth="sm"
            fullWidth
          >
            <DialogTitle>Add Country to Proposal</DialogTitle>
            <DialogContent>
              <FormControl fullWidth sx={{ mt: 2 }}>
                <InputLabel>Country</InputLabel>
                <Select
                  value={newItem.country_name || ""}
                  onChange={(e) => handleCountryChange(e.target.value)}
                  label="Country"
                  MenuProps={{
                    PaperProps: {
                      style: {
                        maxHeight: "200px",
                        overflow: "auto",
                      },
                    },
                  }}
                >
                  {countries.map((country) => (
                    <MenuItem key={country} value={country}>
                      {country}
                    </MenuItem>
                  ))}
                </Select>
              </FormControl>

              <FormControl fullWidth sx={{ mt: 2 }}>
                <InputLabel>Payout Method</InputLabel>
                <Select
                  value={newItem.payout_method}
                  onChange={(e) => handlePayoutMethodChange(e.target.value)}
                  disabled={!newItem.country_code}
                  label="Payout Method"
                >
                  {payoutMethods.map((method) => (
                    <MenuItem key={method} value={method}>
                      {titleCase_(method)}
                    </MenuItem>
                  ))}
                </Select>
              </FormControl>

              <TextField
                fullWidth
                label="Proposed Rate"
                sx={{ mt: 2 }}
                value={newItem.proposed_rate}
                disabled
              />

              {/* Display the duplicate warning */}
              {duplicateWarning && (
                <Alert severity="error" sx={{ mt: 2 }}>
                  {duplicateWarning}
                </Alert>
              )}

              {!duplicateWarning && (
                <TextField
                  fullWidth
                  required
                  label="Reason for Addition"
                  multiline
                  rows={3}
                  sx={{ mt: 2 }}
                  value={newItem.reason}
                  onChange={(e) =>
                    setNewItem((prev) => ({
                      ...prev,
                      reason: e.target.value,
                    }))
                  }
                  error={!newItem.reason}
                  helperText={!newItem.reason ? "Reason is required" : ""}
                />
              )}
            </DialogContent>
            <DialogActions>
              <Button onClick={() => setAddItemModal(false)}>Cancel</Button>
              <Button
                onClick={handleAddItem}
                disabled={
                  !newItem.country_code ||
                  !newItem.payout_method ||
                  !newItem.proposed_rate ||
                  !newItem.reason?.trim() ||
                  !!duplicateWarning
                }
                variant="contained"
              >
                Add Item
              </Button>
            </DialogActions>
          </Dialog>
          <Box sx={{ display: "flex", gap: 2, justifyContent: "flex-end" }}>
            <Button variant="outlined" onClick={() => navigate(-1)}>
              Cancel
            </Button>
            <Button
              variant="contained"
              onClick={handleSubmit}
              disabled={!proposal?.items?.length}
            >
              Submit for Review
            </Button>
          </Box>
        </>
      )}
    </Box>
  );
}
import { useEffect, useCallback, useState, useRef } from "react";
import { useDispatch, useSelector } from "react-redux";
import { useParams, useNavigate, useLocation } from "react-router";
import { Alert } from "@mui/lab";
import { Add } from "@mui/icons-material";
import {
    getProposalByID,
    updateProposalAPI,
    updateProposalStatusAPI,
    updateItemVolumeAPI,
    addProposalItemAPI,
    deleteProposalItemAPI,
} from "@services-(pms)/proposals";
import { request } from "@utils/index";
import { Box, Button, Card, Grid2, IconButton, Stack } from "@mui/material";
import { useMemo } from "react";
import {
    TextBoldBig,
    TextMedium,
    TextNormal,
} from "@/common/Typography/CustomTypography";
import { getFeeRackRate, getFeeRackSpeedTexts } from "@services/feeRackRate";
import BasicDatePicker from "@/common/DateAndTime/DatePicker";
import { renderToaster } from "@utils/notificationUtils";
import CustomModal from "@modules-(pms)/components/CustomModal";
import ItemsTables from "./ItemsTables";
import CommonButton from "@/common/Button/CommonButton";

import {
    Dialog,
    DialogTitle,
    DialogContent,
    DialogActions,
    Typography,
    Grid,
    FormControl,
    InputLabel,
    Select,
    Paper,
    MenuItem,
    TextField,
    Chip,
    Divider,
    List,
    ListItem,
    ListItemText
} from "@mui/material";
import useUserMgmt from "@/hooks/useUserMgmt";
import { hardcodedCountries } from "@modules-(onboarding)/configuration/FeeRackRate/FeeRackRate/defaults";

const DialogType = Object.freeze({
    ALERT: "ALERT",
    CONFIRM: "CONFIRM",
    MSG: "MESSAGE",
});

const ViewProposal = ({ access }) => {
    const { id: proposalId } = useParams();
    const dispatch = useDispatch();
    const navigate = useNavigate();
    const location = useLocation();

    const { Decrypt } = useUserMgmt()

    const userid = useSelector((state) => state.userReducer.userId);
    const sid = useSelector((state) => state.userReducer.profile.sid);
    const dec_id = Decrypt(userid, sid);

    const countryList = useSelector((state) => state.commonReducer?.countryList);
    const partnerList = useSelector((state) => state.commonReducer.partnerList);
    const userList = useSelector((state) => state.commonReducer.userList);
    const refOriginalProposal = useRef({});

    const [filteredSlabs, setFilteredSlabs] = useState([]);
    const [rateBelowMinimumWarning, setRateBelowMinimumWarning] = useState('');
    const [hasAttemptedVolumeChange, setHasAttemptedVolumeChange] = useState(false);

    const filterSlabsByVolume = (slabs, volume, volumeType) => {
        // If no volume provided, don't show any slabs
        if (!volume || isNaN(parseFloat(volume))) {
            setFilteredSlabs([]);
            return;
        }

        const numericVolume = parseFloat(volume);
        console.log(`Filtering slabs for volume: ${numericVolume}, type: ${volumeType}`);

        // Filter slabs that match the volume type and include the committed volume
        const matchingSlabs = slabs.filter(slab => {
            // First make sure we only use slabs that match the selected payout method
            if (newItem.payout_method &&
                slab.payout_type &&
                String(slab.payout_type) !== String(newItem.payout_method)) {
                return false;
            }

            // Match on volume type
            if (slab.volume_type !== volumeType) {
                return false;
            }

            // Check if volume falls within the slab range
            const minValue = parseFloat(slab.min_value) || 0;
            const maxValue = slab.max_value ? parseFloat(slab.max_value) : Infinity;

            const isInRange = numericVolume >= minValue && numericVolume <= maxValue;
            console.log(`Slab ${slab.frr_id}: Range ${minValue}-${maxValue}, Volume ${numericVolume}, isInRange: ${isInRange}`);

            return isInRange;
        });

        console.log("Matching slabs after filtering:", matchingSlabs);

        // If we found a matching slab, select the first one automatically
        if (matchingSlabs.length > 0) {
            handleSelectSlab(matchingSlabs[0]);
        }
        setFilteredSlabs(matchingSlabs);
    };

    const getCountryName = (code) => {
        const other = hardcodedCountries.find(c => c.value === code);
        if (other?.label) {
            return other.label;
        }
        const name = countryList.find(c => c.country_code == code)?.country_name || "";
        return `${name} (${code})`;
    }

    const handleEditVolumeCommitmentChange = (value) => {
        setHasAttemptedVolumeChange(true); // Set flag when user changes value
        setSelectedItemForVolume(prev => ({
            ...prev,
            committed_volume: value
        }));

        // Filter slabs based on new volume
        // filterSlabsForEdit(availableVolumeSlabs, value, selectedItemForVolume.volume_type);
    };

    const handleEditVolumeTypeChange = (volumeType) => {
        setSelectedItemForVolume(prev => ({
            ...prev,
            volume_type: volumeType
        }));

        // Re-filter slabs when volume type changes
        if (selectedItemForVolume.committed_volume) {
            // filterSlabsForEdit(availableVolumeSlabs, selectedItemForVolume.committed_volume, volumeType);
        }
    };

    const [filteredEditSlabs, setFilteredEditSlabs] = useState([]);

    const filterSlabsForEdit = (slabs, volume, volumeType) => {
        // If no volume provided, don't show any slabs
        setRateBelowMinimumWarning('');

        if (!volume || isNaN(parseFloat(volume))) {
            setFilteredEditSlabs([]);
            return;
        }

        const numericVolume = parseFloat(volume);
        console.log(`Filtering edit slabs for volume: ${numericVolume}, type: ${volumeType}`);

        // Filter slabs that match the volume type and include the committed volume
        const matchingSlabs = slabs.filter(slab => {
            // First, filter by payout_type to match the selected item's payout_type
            if (selectedItemForVolume.payout_type &&
                slab.payout_type &&
                parseInt(slab.payout_type) !== parseInt(selectedItemForVolume.payout_type)) {
                return false;
            }
            if (selectedItemForVolume.fee_type) {
                switch (selectedItemForVolume.fee_type) {
                    case "MIXED":
                        if (slab.custom_fee_enable != 1) {
                            return false;
                        }
                        break;
                    case "FLAT":
                        if (slab.calculation_mode != 'flat' || slab.custom_fee_enable == 1) {
                            return false;
                        }
                        break;
                    case "PERCENTAGE":
                        if (slab.calculation_mode != 'percentage' || slab.custom_fee_enable == 1) {
                            return false;
                        }
                        break;
                    default:
                        break;
                }
            }

            // For each slab, get the proper volume type
            const slabVolumeType = slab.velocity_type === 1 ? 'TRANSACTION_COUNT' : 'AMOUNT';

            // Match on volume type
            if (slabVolumeType !== volumeType) {
                return false;
            }

            // Check if volume falls within the slab range
            const minValue = parseFloat(slab.min_value) || 0;
            const maxValue = slab.max_value ? parseFloat(slab.max_value) : Infinity;

            const isInRange = numericVolume >= minValue && numericVolume <= maxValue;
            console.log(`Edit slab: Range ${minValue}-${maxValue}, Volume ${numericVolume}, isInRange: ${isInRange}`);

            setTimeout(() => {
                checkRateBelowMinimum();
            }, 0);

            return isInRange;
        });

        console.log("Matching edit slabs after filtering:", matchingSlabs);

        // If we found a matching slab, select the first one automatically
        if (matchingSlabs.length > 0) {
            const slab = matchingSlabs[0];

            // Handle the fee based on calculation mode
            let feeType = (slab.calculation_mode || 'FLAT').toUpperCase();
            let feeFlat = 0;
            let feePercentage = 0;

            if (feeType === 'FLAT' && slab.custom_fee_enable !== 1) {
                feeFlat = parseFloat(slab.proposed_fee || 0);
            } else if (feeType === 'PERCENTAGE' && slab.custom_fee_enable !== 1) {
                feePercentage = parseFloat(slab.proposed_fee || 0);
            } else if (slab.custom_fee_enable === 1) {
                const customFee = slab.custom_proposed_fee ? JSON.parse(slab.custom_proposed_fee) : null;
                if (feeType === 'PERCENTAGE') {
                    feeFlat = customFee.fees || 0;
                    feePercentage = parseFloat(slab.proposed_fee || 0);
                } else {
                    feeFlat = parseFloat(slab.proposed_fee || 0);
                    feePercentage = customFee.fees || 0;
                }

                /* const parts = String(slab.proposed_fee || '').split('+');
                if (parts.length >= 2) {
                    feeFlat = parseFloat(parts[0].trim()) || 0;
                    feePercentage = parseFloat(parts[1].replace(/%/g, '').trim()) || 0;
                } else {
                    feeFlat = parseFloat(slab.proposed_fee || 0);
                } */
                feeType = 'MIXED';
            }

            // Update selected item with the values from the matching slab
            setSelectedItemForVolume(prev => ({
                ...prev,
                fee_type: feeType,
                fee_flat: feeFlat,
                fee_percentage: feePercentage
            }));
        }
        setFilteredEditSlabs(matchingSlabs);
    };


    const handleVolumeCommitmentChange = (value) => {
        setNewItem(prev => ({
            ...prev,
            committed_volume: value
        }));

        // Filter slabs based on new volume
        filterSlabsByVolume(addCountrySlabs, value, newItem.volume_type);
    };

    const handleVolumeTypeChange = (volumeType) => {
        setNewItem(prev => ({
            ...prev,
            volume_type: volumeType
        }));

        // Re-filter slabs when volume type changes
        if (newItem.committed_volume) {
            filterSlabsByVolume(addCountrySlabs, newItem.committed_volume, volumeType);
        }
    };

    // Add this function to check if the entered rate is below the minimum
    const checkRateBelowMinimum = () => {
        // If no slabs are selected or available, we can't check
        if (!filteredEditSlabs.length) return;

        // Get the matching slab
        const matchingSlab = filteredEditSlabs?.[0];

        // Extract the minimum fee from the slab (minimum_fee field)
        const minimumFee = parseFloat(matchingSlab.minimum_fee || 0);

        let isBelow = false;
        let warningMessage = '';

        // Compare based on fee type
        if (selectedItemForVolume.fee_type === 'FLAT') {
            const flatRate = parseFloat(selectedItemForVolume.fee_flat || 0);
            if (flatRate < minimumFee) {
                isBelow = true;
                warningMessage = `Warning: The flat fee ${flatRate.toFixed(2)} USD is below the minimum rate of ${minimumFee.toFixed(2)} USD for this slab.`;
            }
        } else if (selectedItemForVolume.fee_type === 'PERCENTAGE') {
            const percentRate = parseFloat(selectedItemForVolume.fee_percentage || 0);
            if (percentRate < minimumFee) {
                isBelow = true;
                warningMessage = `Warning: The percentage fee ${percentRate.toFixed(2)}% is below the minimum rate of ${minimumFee.toFixed(2)}% for this slab.`;
            }
        } else if (selectedItemForVolume.fee_type === 'MIXED') {
            // For mixed fees, typically check the flat component
            const flatRate = parseFloat(selectedItemForVolume.fee_flat || 0);
            if (flatRate < minimumFee) {
                isBelow = true;
                warningMessage = `Warning: The flat component ${flatRate.toFixed(2)} USD is below the minimum rate of ${minimumFee.toFixed(2)} USD for this slab.`;
            }
        }

        setRateBelowMinimumWarning(warningMessage);
        return isBelow;
    };

    // Modify the handlers for fee flat change
    const handleFeeRateChange = (value, type) => {
        if (type === 'flat') {
            setSelectedItemForVolume(prev => ({
                ...prev,
                fee_flat: value
            }));
        } else {
            setSelectedItemForVolume(prev => ({
                ...prev,
                fee_percentage: value
            }));
        }

        // Use a setTimeout to ensure state has updated before check
        /* setTimeout(() => {
            checkRateBelowMinimum();
        }, 0); */
    };

    const [feeRack, setFeeRack] = useState([]);
    const [countryWiseData, setCountryWiseData] = useState({});

    const [error, setError] = useState("");
    const [edit, setEdit] = useState(location?.state?.edit || false);
    const [dialogState, setDialogState] = useState({
        open: false,
        type: null,
        proposalId: null,
        comments: "",
    });
    const [proposal, setProposal] = useState({
        loading: true,
    });
    const [paymentInstruments, setPaymentInstruments] = useState([]);
    const [dialog, setDialog] = useState({
        open: false,
        type: "",
        action: () => { },
        msg: "",
    });

    const [addCountrySlabs, setAddCountrySlabs] = useState([]);

    // Add Country Modal State - moved inside component
    const [addCountryModal, setAddCountryModal] = useState(false);
    const [countries, setCountries] = useState([]);
    const [payoutMethods, setPayoutMethods] = useState([]);
    const [newItem, setNewItem] = useState({
        country_code: '',
        country_name: '',
        payout_method: '',
        proposed_rate: '',
        reason: '',
        volume_type: 'AMOUNT'
    });
    const [duplicateWarning, setDuplicateWarning] = useState('');

    // Edit Volume Modal State
    const [editVolumeModal, setEditVolumeModal] = useState(false);
    const [selectedItemForVolume, setSelectedItemForVolume] = useState({
        id: null,
        country_code: '',
        payout_type: '',
        payout_method: '',
        committed_volume: 0,
        fee_type: 'FLAT',
        fee_flat: 0,
        fee_percentage: 0,
        volume_type: 'AMOUNT'
    });
    const [availableVolumeSlabs, setAvailableVolumeSlabs] = useState([]);

    const availableVolumeTypes = useMemo(() => {
        const volumes = [];
        availableVolumeSlabs?.forEach((slab) => {
            if (!volumes.find((vol) => vol.value == slab.velocity_period)) {
                volumes.push({
                    value: slab.velocity_period,
                    option: slab.velocity_period == 1 ? "TRANSACTION_COUNT" : "AMOUNT",
                    label: slab.velocity_period == 1 ? "Transaction Count Based" : "Amount Based",
                });
            }
        });
        return volumes;
    }, [availableVolumeSlabs]);

    const titleCase_ = (str) => {
        if (!str || typeof str !== 'string') return '';

        return str
            .toLowerCase()
            .split('_')
            .map(word => word.charAt(0).toUpperCase() + word.slice(1))
            .join(' ');
    };

    // In View.jsx, modify the handleDeleteItem function:
    const handleDeleteItem = async (item, index) => {
        console.log("handleDeleteItem called with:", item, index);
        try {
            // Set the dialog state directly to open the confirmation
            setDialog({
                open: true,
                type: DialogType.CONFIRM,
                action: async () => {
                    try {
                        dispatch(addLoader("DELETE_ITEM"));
                        console.log("Calling request with:", {
                            api: deleteProposalItemAPI,
                            params: { id: item.id || item.item_id }
                        });

                        // Make the API request
                        const response = await request({
                            api: deleteProposalItemAPI,
                            params: { id: item.id || item.item_id },
                        });

                        console.log("API response:", response);

                        if (response.status === 200) {
                            renderToaster({
                                type: "success",
                                message: "Item deleted successfully"
                            });

                            // Refresh proposal data after deletion
                            await getProposalById();
                        } else {
                            renderToaster({
                                type: "error",
                                message: response.data?.message || "Failed to delete item"
                            });
                        }
                    } catch (error) {
                        console.error("Error in API call:", error);
                        renderToaster({
                            type: "error",
                            message: "Error deleting item: " + (error.message || "Unknown error")
                        });
                    } finally {
                        dispatch(removeLoader("DELETE_ITEM"));
                    }
                },
                msg: "Are you sure you want to delete this item?",
                title: "Confirm Delete",
            });
            console.log("Dialog state set:", dialog);
        } catch (error) {
            console.error("Error setting up delete dialog:", error);
        }
    };

    // Helper function to get payout method name
    const getPayoutMethodName = (payoutTypeId) => {
        if (!payoutTypeId) return 'Unknown';
        if (!paymentInstruments || paymentInstruments.length === 0) {
            return `Payout Type ${payoutTypeId}`;
        }

        const paymentMethod = paymentInstruments.find(method =>
            String(method.value) === String(payoutTypeId)
        );
        return paymentMethod?.label || `Payout Type ${payoutTypeId}`;
    };

    const fetchCountriesAndMethods = async () => {
        try {
            dispatch(addLoader("FETCH_COUNTRIES"));
            console.log("Fetching available countries...");

            // Replace with your actual API endpoint
            const response = await request({
                api: getFeeRackRate,
                params: `status=ACTIVE`,
            });

            console.log("Countries API response:", response);

            if (response.status === 200) {
                // Extract unique countries from the fee rack rates
                const uniqueCountries = [...new Set(response.data.data.map(item => item.country_code))]?.sort((a, b) => a.localeCompare(b));
                console.log("Unique countries found:", uniqueCountries);
                setCountries(uniqueCountries || []);
            }
        } catch (error) {
            console.error("Error fetching countries:", error);
            setError("Failed to fetch countries");
        } finally {
            dispatch(removeLoader("FETCH_COUNTRIES"));
        }
    };

    // Add handleCountryChange function
    const handleCountryChange = async (country) => {
        try {
            setDuplicateWarning('');
            setError('');

            // Fetch fee rack rates specifically for this country
            dispatch(addLoader("FETCH_COUNTRY_RATES"));
            const response = await request({
                api: getFeeRackRate,
                params: `&country=${country}`,
            });

            if (response.status === 200) {
                // Filter out DELETED records before storing and processing
                const activeFeeRackRates = response.data.data.filter(item => item.status !== 'DELETED');

                // Store the filtered fee rack rates for this country
                setFeeRack(activeFeeRackRates);
                console.log(`Active fee rack rates for ${country}:`, activeFeeRackRates);

                // Extract unique payout_type IDs for this country (only from active records)
                const payoutTypeIds = [...new Set(activeFeeRackRates.map(item => item.payout_type))];

                // Now fetch payment instrument details
                const paymentResponse = await request({
                    api: getFeeRackSpeedTexts,
                });

                // if (paymentResponse.status === 200) {
                //     // Filter payment instruments to only show ones used in this country
                //     const availablePaymentMethods = paymentResponse.data.data2
                //         .filter(item => payoutTypeIds.includes(item.payment_instrument_id))
                //         .map(item => ({
                //             id: item.payment_instrument_id,
                //             name: titleCase_(item.type || item.payment_instrument_name)
                //         }));

                //     setPayoutMethods(availablePaymentMethods);
                // } else {
                //     // Fallback: just use the numeric IDs
                //     setPayoutMethods(payoutTypeIds.map(id => ({ id, name: `Type ${id}` })));
                // }

                if (paymentResponse.status === 200) {
                    // Filter payment instruments to only show ones used in this country
                    const allPaymentMethods = paymentResponse.data.data2
                        .filter(item => payoutTypeIds.includes(item.payment_instrument_id))
                        .map(item => ({
                            id: item.payment_instrument_id,
                            name: titleCase_(item.type || item.payment_instrument_name)
                        }));

                    // Filter out already used payout methods for this country
                    const availablePaymentMethods = getAvailablePayoutMethods(country, allPaymentMethods);

                    setPayoutMethods(availablePaymentMethods);

                    if (availablePaymentMethods.length === 0 && allPaymentMethods.length > 0) {
                        setDuplicateWarning('All available payout methods for this country are already in the proposal');
                    }
                } else {
                    // Fallback: just use the numeric IDs
                    const allPaymentMethods = payoutTypeIds.map(id => ({ id, name: `Type ${id}` }));
                    const availablePaymentMethods = getAvailablePayoutMethods(country, allPaymentMethods);
                    setPayoutMethods(availablePaymentMethods);

                    if (availablePaymentMethods.length === 0 && allPaymentMethods.length > 0) {
                        setDuplicateWarning('All available payout methods for this country are already in the proposal');
                    }
                }
            } else {
                setError(`No fee rack rates found for country ${country}`);
                setPayoutMethods([]);
            }

            setNewItem(prev => ({
                ...prev,
                country_code: country,
                country_name: country,
                payout_method: '',
                proposed_rate: ''
            }));
        } catch (error) {
            console.error('Error in handleCountryChange:', error);
            setError('Failed to fetch payout methods. Please try again.');
            setPayoutMethods([]);
        } finally {
            dispatch(removeLoader("FETCH_COUNTRY_RATES"));
        }
    };

    const handlePayoutMethodChange = async (payoutMethodId) => {
        try {
            setDuplicateWarning('');

            if (newItem.country_code && payoutMethodId) {
                const normalizedPayoutId = String(payoutMethodId);
                const isDuplicate = proposal.items && proposal.items.some(
                    (item) => {
                        if (!item || !item.country_code) return false;
                        const countryMatch = item.country_code.toLowerCase() === newItem.country_code.toLowerCase();
                        const itemPayoutId = String(item.payout_type || item.payout_method || '');
                        const payoutMatch = itemPayoutId === normalizedPayoutId;
                        // const payoutMatch =
                        //     (item.payout_type && payoutMethodId && item.payout_type.toString() === payoutMethodId.toString()) ||
                        //     (item.payout_method && payoutMethodId && item.payout_method.toString() === payoutMethodId.toString());

                        return countryMatch && payoutMatch;
                    }
                );

                if (isDuplicate) {
                    setDuplicateWarning('This country with this payout method already exists in this proposal');
                    // Return early so we don't proceed with this selection
                    setNewItem(prev => ({
                        ...prev,
                        payout_method: ''
                    }));
                    return;
                }

                // Find the matching fee rack rate for this combination
                const matchingRates = feeRack.filter(rate =>
                    rate.payout_type?.toString() === payoutMethodId?.toString()
                );

                console.log("Selected payout method:", payoutMethodId);
                console.log("Matching rates for this payout method:", matchingRates);

                if (matchingRates.length === 0) {
                    setError(`No fee rack rate found for payout method ${payoutMethodId} in country ${newItem.country_code}`);
                    setAddCountrySlabs([]); // Clear slabs if none found
                } else {
                    setError(''); // Clear errors if we found a match

                    // Format the slabs for display but don't show them yet
                    const formattedSlabs = matchingRates.map(rate => {
                        // Handle the fee based on calculation mode
                        let feeType = (rate.calculation_mode || 'FLAT').toUpperCase();
                        let feeFlat = 0;
                        let feePercentage = 0;

                        console.log(`Processing rate with calculation_mode: ${rate.calculation_mode}, proposed_fee: ${rate.proposed_fee}`);

                        if (feeType === 'FLAT') {
                            // For FLAT fee type, parse the proposed_fee
                            feeFlat = parseFloat(rate.proposed_fee || 0);
                            console.log(`Parsed FLAT fee: ${rate.proposed_fee} -> ${feeFlat}`);
                        } else if (feeType === 'PERCENTAGE') {
                            // For PERCENTAGE fee type, parse the proposed_fee
                            feePercentage = parseFloat(rate.proposed_fee || 0);
                            console.log(`Parsed PERCENTAGE fee: ${rate.proposed_fee} -> ${feePercentage}`);
                        } else if (feeType === 'MIXED') {
                            // For MIXED, try to parse both parts
                            const parts = String(rate.proposed_fee || '').split('+');
                            console.log(`Trying to parse MIXED fee: ${rate.proposed_fee}, parts:`, parts);

                            if (parts.length >= 2) {
                                feeFlat = parseFloat(parts[0].trim()) || 0;
                                feePercentage = parseFloat(parts[1].replace(/%/g, '').trim()) || 0;
                                console.log(`Parsed MIXED fee parts: flat=${feeFlat}, percentage=${feePercentage}`);
                            } else {
                                // If no + sign, treat as flat fee
                                feeFlat = parseFloat(rate.proposed_fee || 0);
                                console.log(`Treated as flat fee: ${feeFlat}`);
                            }
                        }

                        return {
                            min_value: rate.min_value || 0,
                            max_value: rate.max_value,
                            volume_type: rate.velocity_type === 1 ? 'TRANSACTION_COUNT' : 'AMOUNT',
                            time_period: rate.velocity_period ?
                                (['DAY', 'WEEK', 'MONTH', 'YEAR'][rate.velocity_period - 1] || 'MONTH') : 'MONTH',
                            fee_type: feeType,
                            fee_flat: feeFlat,
                            fee_percentage: feePercentage,
                            frr_id: rate.frr_id,
                            original_proposed_fee: rate.proposed_fee // Keep original for debugging
                        };
                    });

                    console.log("All available slabs:", formattedSlabs);

                    // Store all slabs but don't display them yet - we'll filter when volume is entered
                    setAddCountrySlabs(formattedSlabs);

                    // If user already entered a volume, filter slabs immediately
                    if (newItem.committed_volume) {
                        filterSlabsByVolume(formattedSlabs, newItem.committed_volume, newItem.volume_type);
                    }
                }
            }

            // Update the selected payout method
            setNewItem(prev => ({
                ...prev,
                payout_method: payoutMethodId
            }));
        } catch (error) {
            console.error('Error in handlePayoutMethodChange:', error);
            setError('Failed to update payout method');
        }
    };

    // Add this function to handle selecting a slab rate
    const handleSelectSlab = (slab) => {
        console.log("Selected slab:", slab);
        setNewItem(prev => ({
            ...prev,
            fee_rack_rate_id: slab.frr_id,
            fee_type: slab.fee_type,
            fee_flat: slab.fee_flat,
            fee_percentage: slab.fee_percentage,
            proposed_rate: formatFeeDisplay(slab.fee_type, slab.fee_flat, slab.fee_percentage),
            volume_type: slab.volume_type
        }));
        console.log("Updated newItem after slab selection:", newItem);
    };

    const addLoader = (id) => {
        return { type: 'ADD_LOADER', payload: id };
    };

    const removeLoader = (id) => {
        return { type: 'REMOVE_LOADER', payload: id };
    };

    const getProposalById = useCallback(async () => {
        // Check if dispatch and addLoader are available
        if (!dispatch) {
            console.error("Dispatch is not available");
            return;
        }

        try {
            dispatch(addLoader("ACTION"));
            const response = await request({
                api: getProposalByID,
                params: { id: proposalId },
            });
            if (response.status == 200) {
                const partnerData = partnerList?.find(
                    (p) => p.partner_id == response.data?.data?.partner_id
                );

                const country_codes = response.data?.data?.items?.map(item =>
                    item.fee_rack_rate?.country_code
                ).filter(Boolean) || [];

                const items = response.data?.data?.items?.map((data, index) => {

                    let isMixedRate = data.custom_fee_enable === 1 || data.fee_rack_rate?.custom_fee_enable === 1;
                    const calculationMode = data.calculation_mode || data.fee_rack_rate?.calculation_mode || 'flat';

                    let processedData = { ...data };

                    isMixedRate = isMixedRate || (data.fee > 0 && data.fee_percentage > 0);

                    if (isMixedRate) {
                        // For mixed rates, extract both components
                        let primaryValue = parseFloat(data.fee || 0);
                        let secondaryValue = parseFloat(data.fee_percentage || 0);

                        // Parse secondary component from custom_proposed_fee JSON
                        // if (data.fee_rack_rate?.custom_proposed_fee || data.custom_proposed_fee) {
                        //     try {
                        //         const customFeeData = JSON.parse(data.fee_rack_rate?.custom_proposed_fee || data.custom_proposed_fee);
                        //         secondaryValue = parseFloat(customFeeData.fees || 0);
                        //     } catch (e) {
                        //         console.log("Failed to parse custom fee JSON", e);
                        //         secondaryValue = 0;
                        //     }
                        // }

                        // Set fee components based on calculation mode
                        // if (calculationMode === 'percentage') {
                        //     // Primary is percentage, secondary is flat
                        //     processedData.fee_percentage = primaryValue;
                        //     processedData.fee_flat = secondaryValue;
                        //     processedData.fee = primaryValue; // Keep original for backward compatibility
                        // } else {
                        // Primary is flat, secondary is percentage  
                        processedData.fee_flat = primaryValue;
                        processedData.fee_percentage = secondaryValue;
                        processedData.fee = primaryValue; // Keep original for backward compatibility
                        // }

                        // Set fee_type to MIXED for proper display
                        processedData.fee_type = "MIXED";
                        processedData.custom_fee_enable = 1;
                    } else {
                        // For non-mixed rates, set values based on calculation mode
                        const feeValue = parseFloat(data.fee || 0);
                        if (calculationMode === 'percentage') {
                            processedData.fee_percentage = feeValue;
                            processedData.fee_flat = 0;
                        } else {
                            processedData.fee_flat = feeValue;
                            processedData.fee_percentage = 0;
                        }
                        processedData.fee_type = calculationMode.toUpperCase();
                        processedData.custom_fee_enable = 0;
                    }

                    // Add an index for easier reference
                    return {
                        ...processedData,
                        oi: index, // Add original index to help with rendering
                        committed_volume: parseFloat(
                            data.committed_volume || data.comitted_volume || data.commited_volume || 0
                        ).toFixed(2),
                        country_code: data.fee_rack_rate?.country_code || data.country_code || "N/A",
                        payout_type: data.fee_rack_rate?.payout_type || data.payout_type || "N/A",
                        payout_currency: data.fee_rack_rate?.payout_currency || data.payout_currency || "N/A",
                        calculation_mode: calculationMode,
                        min_value: data.min_value || data.fee_rack_rate?.min_value || 0,
                        max_value: data.max_value || data.fee_rack_rate?.max_value || 0,
                    };

                });
                let proposal = response.data?.data;
                proposal.items = items;
                console.log("Received Proposal Data:", proposal);

                setProposal({
                    ...(proposal || {}),
                    partner: partnerData,
                    loading: false,
                    country_codes
                });

                refOriginalProposal.current = {
                    ...JSON.parse(JSON.stringify(response.data.data || {})),
                    partner: partnerData,
                    loading: false,
                    country_codes
                };
            } else {
                setProposal({ loading: false });
            }
        } catch (error) {
            console.error("Error fetching proposal:", error);
            setProposal({ loading: false, error: true });
            setError("Failed to load proposal data");
        } finally {
            dispatch(removeLoader("ACTION"));
        }
    }, [proposalId, dispatch, partnerList]);

    const handleCountrySelection = (itemId) => {
        const item = proposal.items.find(i => i.item_id === itemId || i.id === itemId);
        setSelectedItemForVolume(item);
        if (item && item.country_code && (item.payout_type || item.payout_method)) {
            fetchAvailableSlabs(item.country_code, item.payout_type || item.payout_method);
        }
    };

    const handleAddItem = async () => {
        try {
            if (!newItem.country_code || !newItem.payout_method) {
                setError('Please select a country and payout method');
                return;
            }

            console.log("Current proposal items before adding:",
                proposal.items ? proposal.items.map(item => ({
                    country: item.country_code,
                    payoutType: item.payout_type,
                    payoutMethod: item.payout_method
                })) : 'No items');

            // Check for duplicates again as a safety measure
            const isDuplicate = proposal.items && proposal.items.some(
                (item) => {
                    if (!item || !item.country_code) return false;

                    const countryMatch = item.country_code.toLowerCase() === newItem.country_code.toLowerCase();
                    const payoutMatch =
                        (item.payout_type && item.payout_type.toString() === newItem.payout_method.toString()) ||
                        (item.payout_method && item.payout_method.toString() === newItem.payout_method.toString());

                    return countryMatch && payoutMatch;
                }
            );

            if (isDuplicate) {
                console.log("DUPLICATE DETECTED!");
                setDuplicateWarning('This country with this payout method already exists in this proposal');
                return;
            }

            // Find matching rack rate
            let matchingRackRate;

            if (newItem.fee_rack_rate_id) {
                // If user manually selected a rate
                matchingRackRate = feeRack.find(rate =>
                    rate.frr_id === newItem.fee_rack_rate_id
                );
                console.log("Using manually selected rack rate:", matchingRackRate);
            } else {
                // Find based on payout method
                const matchingRates = feeRack.filter(rate =>
                    rate.payout_type?.toString() === newItem.payout_method?.toString()
                );

                console.log("Found matching rates for payout method:", matchingRates);

                if (matchingRates.length > 0) {
                    // Sort by min_value (descending) to get the most specific slab
                    matchingRackRate = matchingRates.sort((a, b) =>
                        (b.min_value || 0) - (a.min_value || 0)
                    )[0];
                }
            }

            console.log("Selected country:", newItem.country_code);
            console.log("Selected payout method:", newItem.payout_method);
            console.log("Matching rack rate:", matchingRackRate);

            if (!matchingRackRate) {
                setError(`No fee rack rate found for ${newItem.country_code} with payout method ${newItem.payout_method}`);
                return;
            }

            // Determine fee values based on fee type - normalize to uppercase for comparison
            const feeType = (newItem.fee_type || matchingRackRate.calculation_mode || 'FLAT').toUpperCase();

            // Parse fees carefully to avoid NaN
            let feeValue = 0;
            let feePercentage = 0;

            console.log("Processing fee with type:", feeType);
            console.log("Proposed fee from rack rate:", matchingRackRate.proposed_fee);

            if (feeType === 'FLAT') {
                // For FLAT fee type, use the proposed_fee directly
                feeValue = parseFloat(matchingRackRate.proposed_fee || 0);
                console.log("Parsed FLAT fee value:", feeValue);
            } else if (feeType === 'PERCENTAGE') {
                // For PERCENTAGE fee type
                feePercentage = parseFloat(matchingRackRate.proposed_fee || 0);
                console.log("Parsed PERCENTAGE fee value:", feePercentage);
            } else if (feeType === 'MIXED') {
                // For MIXED, we need both values
                const parts = String(matchingRackRate.proposed_fee || '').split('+');
                if (parts.length >= 2) {
                    feeValue = parseFloat(parts[0].trim()) || 0;
                    feePercentage = parseFloat(parts[1].replace('%', '').trim()) || 0;
                    console.log("Parsed MIXED fee values:", { flat: feeValue, percentage: feePercentage });
                }
            }

            // Check if we have any values from newItem (from manual selection)
            if (newItem.fee_flat !== undefined && !isNaN(parseFloat(newItem.fee_flat))) {
                feeValue = parseFloat(newItem.fee_flat);
                console.log("Using manually selected flat fee:", feeValue);
            }

            if (newItem.fee_percentage !== undefined && !isNaN(parseFloat(newItem.fee_percentage))) {
                feePercentage = parseFloat(newItem.fee_percentage);
                console.log("Using manually selected percentage fee:", feePercentage);
            }

            // Ensure we have valid numbers
            if (isNaN(feeValue)) feeValue = 0;
            if (isNaN(feePercentage)) feePercentage = 0;

            console.log("Final fee values to send:", {
                feeType,
                feeValue,
                feePercentage
            });

            // Prepare payload with the field names expected by the backend
            const payload = {
                proposal_id: proposalId,
                fee_rack_rate_id: matchingRackRate.frr_id,
                fee: feeValue, // This is what the backend expects
                fee_percentage: feePercentage,
                negotiation_level: 0,
                item_version: 1.0,
                negotiation_count: 0,
                min_value: matchingRackRate.min_value || 0,
                max_value: matchingRackRate.max_value || null,
                velocity_type: matchingRackRate.velocity_type || 2,
                velocity_period: matchingRackRate.velocity_period || 2,
                userId: dec_id || 'system',
                commited_volume: newItem.committed_volume ? parseFloat(newItem.committed_volume) : null
            };

            console.log("Sending payload:", payload);

            dispatch(addLoader("ADD_ITEM"));
            const addResponse = await request({
                api: addProposalItemAPI,
                params: { id: proposalId }, // This is needed for the API function
                payload: payload
            });

            if (addResponse.data) {
                renderToaster({
                    type: "success",
                    message: "Item added successfully"
                });

                // Perform auto-approval check for this new item
                // Step 1: Check if we have the minimum fee values from the rack rate
                const walkawayFeeFlat = parseFloat(matchingRackRate.minimum_fee || 0);
                const isAutoApprovalEligible = checkAutoApprovalEligibility(
                    feeType,
                    feeValue,
                    feePercentage,
                    walkawayFeeFlat
                );

                console.log("Auto-approval check:", {
                    feeType,
                    feeValue,
                    feePercentage,
                    walkawayFeeFlat,
                    isEligible: isAutoApprovalEligible
                });

                // Step 2: Update proposal status based on eligibility if needed
                if (!isAutoApprovalEligible && proposal.proposal_status === 'APPROVED') {
                    console.log("Updating proposal status to PENDING_APPROVAL because the new item is below minimum rates");

                    // Call API to update proposal status
                    try {
                        await request({
                            api: updateProposalStatusAPI,
                            payload: {
                                proposal_id: Number(proposalId),
                                new_status: "PENDING_APPROVAL",
                                comments: "Automatically changed to pending approval due to new item with below minimum rates"
                            },
                        });

                        console.log("Successfully updated proposal status to PENDING_APPROVAL");
                    } catch (statusError) {
                        console.error("Error updating proposal status:", statusError);
                    }
                }

                setAddCountryModal(false);
                setNewItem({
                    country_code: '',
                    country_name: '',
                    payout_method: '',
                    proposed_rate: '',
                    reason: '',
                    committed_volume: ''
                });
                setAddCountrySlabs([]);

                await getProposalById();
            }
        } catch (error) {
            console.error('Error adding item:', error);
            setError(error.response?.data?.error || 'Failed to add item');
        } finally {
            dispatch(removeLoader("ADD_ITEM"));
        }
    };

    // Modify the getAvailablePayoutMethods function to look deeper into the proposal items
    const getAvailablePayoutMethods = (country, allPaymentMethods) => {
        if (!proposal?.items || !country) return allPaymentMethods;

        console.log("DETAILED PROPOSAL ITEMS:", proposal.items.map(item => ({
            id: item.id || item.item_id,
            country: item.country_code,
            countryFromRackRate: item.fee_rack_rate?.country_code,
            payoutType: item.payout_type,
            payoutMethod: item.payout_method,
            payoutTypeFromRackRate: item.fee_rack_rate?.payout_type,
            fullItem: item
        })));

        // Find which payout methods are already used for this country
        const usedPayoutMethodIds = [];

        proposal.items.forEach(item => {
            // Get the country code from either direct property or nested fee_rack_rate
            const itemCountry = item.country_code || item.fee_rack_rate?.country_code;

            if (item && itemCountry?.toLowerCase() === country.toLowerCase()) {
                // Get the payout ID from any possible location
                const methodId = item.payout_type || item.payout_method || item.fee_rack_rate?.payout_type;

                if (methodId) {
                    // Always normalize to string format
                    const normalizedId = String(methodId);
                    usedPayoutMethodIds.push(normalizedId);

                    console.log(`Found used method: ID=${normalizedId} (${getPayoutMethodName(normalizedId)}) for ${country}`);
                }
            }
        });

        console.log("Used payout method IDs for", country, ":", usedPayoutMethodIds);

        // Filter out the already used payout methods by ID
        const availableMethods = allPaymentMethods.filter(method => {
            const normalizedMethodId = String(method.id);
            const isAvailable = !usedPayoutMethodIds.includes(normalizedMethodId);

            console.log(`Method ${normalizedMethodId} (${method.name}) available: ${isAvailable}`);
            return isAvailable;
        });

        console.log("Available methods after filtering:", availableMethods.map(m => `${m.id} (${m.name})`));

        return availableMethods;
    };

    // At the top of your component, add a derived state that checks for duplicates
    const isDuplicateItem = useMemo(() => {
        if (!newItem.country_code || !newItem.payout_method || !proposal.items) {
            return false;
        }

        return proposal.items.some(item => {
            if (!item) return false;

            // Get country from either direct property or nested fee_rack_rate
            const itemCountry = item.country_code || item.fee_rack_rate?.country_code;
            if (!itemCountry) return false;

            const countryMatch = itemCountry.toLowerCase() === newItem.country_code.toLowerCase();

            // Get payout ID from any possible location
            const itemPayoutId = item.payout_type || item.payout_method || item.fee_rack_rate?.payout_type;
            if (!itemPayoutId) return false;

            const newItemPayoutId = String(newItem.payout_method || '');
            const payoutMatch = String(itemPayoutId) === newItemPayoutId;

            return countryMatch && payoutMatch;
        });
    }, [newItem.country_code, newItem.payout_method, proposal.items]);

    // Then in your useEffect, set the warning message whenever the duplicate status changes
    useEffect(() => {
        if (isDuplicateItem) {
            setDuplicateWarning('This country with this payout method already exists in this proposal');
        } else {
            setDuplicateWarning('');
        }
    }, [isDuplicateItem]);

    // Helper function for auto-approval eligibility check
    const checkAutoApprovalEligibility = (feeType, feeValue, feePercentage, walkawayFeeFlat) => {
        // Based on the logic in proposalEligibility.js
        if (feeType === 'FLAT' && feeValue < walkawayFeeFlat) {
            console.log(`Flat fee ${feeValue} is below minimum ${walkawayFeeFlat}`);
            return false;
        }

        if (feeType === 'PERCENTAGE' && feePercentage < walkawayFeeFlat) {
            console.log(`Percentage fee ${feePercentage} is below minimum ${walkawayFeeFlat}`);
            return false;
        }

        if (feeType === 'MIXED') {
            // For mixed, we'd need more complex logic here based on your actual requirements
            // This is a simplified version:
            if (feeValue < walkawayFeeFlat) {
                console.log(`Mixed fee flat component ${feeValue} is below minimum ${walkawayFeeFlat}`);
                return false;
            }
        }

        return true;
    };

    // Add this function to handle expiry date updates
    const handleExpiryDateChange = async (newDate) => {
        // Check if the selected date is valid (not today or in the past)
        const today = new Date();
        today.setHours(0, 0, 0, 0); // Reset time to start of day for proper comparison

        const selectedDate = new Date(newDate);
        selectedDate.setHours(0, 0, 0, 0);

        if (selectedDate <= today) {
            setError("Expiry date must be in the future");
            return;
        }

        try {
            // Set the new date in the local state
            setProposal((prev) => ({
                ...prev,
                new_expiry_date: new Date(newDate),
                expiry_date: new Date(newDate) // Also update the original expiry_date
            }));

            // Update the database
            dispatch(addLoader("UPDATE_EXPIRY"));

            const response = await request({
                api: updateProposalAPI,
                payload: {
                    id: proposalId,
                    expiry_date: new Date(newDate).toISOString(),
                },
                params: { id: proposalId },
            });

            if (["200", "201"].includes(response?.status?.toString())) {
                renderToaster({
                    type: "success",
                    message: "Expiry date updated successfully",
                });

                // Update the reference copy to sync with the new date
                refOriginalProposal.current = {
                    ...refOriginalProposal.current,
                    expiry_date: new Date(newDate),
                    new_expiry_date: new Date(newDate)
                };
            } else {
                // If update failed, revert to original expiry date
                setProposal((prev) => ({
                    ...prev,
                    new_expiry_date: new Date(refOriginalProposal.current.expiry_date),
                    expiry_date: new Date(refOriginalProposal.current.expiry_date)
                }));
                setError("Failed to update expiry date");
            }
        } catch (error) {
            console.error("Error updating expiry date:", error);
            setError("Failed to update expiry date: " + (error.message || "Unknown error"));

            // Revert to original expiry date on error
            setProposal((prev) => ({
                ...prev,
                new_expiry_date: new Date(refOriginalProposal.current.expiry_date),
                expiry_date: new Date(refOriginalProposal.current.expiry_date)
            }));
        } finally {
            dispatch(removeLoader("UPDATE_EXPIRY"));
        }
    };

    // Add a function to check if a date is valid (in the future)
    const isValidExpiryDate = (date) => {
        const today = new Date();
        today.setHours(0, 0, 0, 0);

        const checkDate = new Date(date);
        checkDate.setHours(0, 0, 0, 0);

        return checkDate > today;
    };

    // const handleSlabUpdate = async () => {
    //     try {
    //         if (!selectedItemForVolume || !selectedItemForVolume.id) {
    //             setError("No valid item selected for update");
    //             return;
    //         }

    //         setError(null);
    //         dispatch(addLoader("UPDATE_SLAB"));

    //         // Find the original item in the proposal items to get the fee_rack_rate_id
    //         const originalItem = proposal.items?.find(
    //             item => item.id === selectedItemForVolume.id || item.item_id === selectedItemForVolume.id
    //         );

    //         if (!originalItem || !originalItem.fee_rack_rate_id) {
    //             console.error("Cannot find fee_rack_rate_id for the selected item");
    //             setError("Missing required data for update. Please try again.");
    //             dispatch(removeLoader("UPDATE_SLAB"));
    //             return;
    //         }

    //         console.log("Current selected item for volume:", selectedItemForVolume);

    //         // Prepare the payload with all necessary fields from the controller
    //         const updatedItemData = {
    //             item_id: selectedItemForVolume.id || selectedItemForVolume.item_id,
    //             proposal_id: proposalId,
    //             fee_rack_rate_id: originalItem.fee_rack_rate_id,
    //             fee: selectedItemForVolume.fee_type === 'FLAT' ? parseFloat(selectedItemForVolume.fee_flat) : 0,
    //             fee_percentage: selectedItemForVolume.fee_type === 'PERCENTAGE' || selectedItemForVolume.fee_type === 'MIXED'
    //                 ? parseFloat(selectedItemForVolume.fee_percentage)
    //                 : 0,
    //             negotiation_level: originalItem.negotiation_level || 0,
    //             item_version: originalItem.item_version || 1.0,
    //             negotiation_count: originalItem.negotiation_count || 0,
    //             min_value: originalItem.min_value || originalItem.slab_from || 0,
    //             max_value: originalItem.max_value || originalItem.slab_to || null,
    //             velocity_type: selectedItemForVolume.volume_type === 'AMOUNT' ? 1 : 2,
    //             velocity_period: originalItem.velocity_period || 2,
    //             userId: JSON.parse(localStorage.getItem('user'))?.username || 'system',
    //             commited_volume: selectedItemForVolume.committed_volume
    //                 ? parseFloat(selectedItemForVolume.committed_volume)
    //                 : 0
    //         };

    //         console.log("Updating item with data:", updatedItemData);

    //         const response = await request({
    //             api: updateItemVolumeAPI,
    //             params: { id: selectedItemForVolume.id || selectedItemForVolume.item_id },
    //             payload: updatedItemData
    //         });

    //         if (response.status === 200) {
    //             renderToaster({
    //                 type: "success",
    //                 message: "Volume commitment and rate updated successfully"
    //             });
    //             await getProposalById();
    //             setEditVolumeModal(false);
    //             setSelectedItemForVolume({
    //                 id: null,
    //                 country_code: '',
    //                 payout_type: '',
    //                 payout_method: '',
    //                 committed_volume: 0,
    //                 fee_type: 'FLAT',
    //                 fee_flat: 0,
    //                 fee_percentage: 0,
    //                 volume_type: 'AMOUNT'
    //             });
    //         } else {
    //             setError((response.data && response.data.message) || 'Failed to update volume commitment and rate');
    //         }
    //     } catch (error) {
    //         console.error('Error updating volume and rate:', error);
    //         setError(error.response?.data?.message || 'Failed to update volume commitment and rate');
    //     } finally {
    //         dispatch(removeLoader("UPDATE_SLAB"));
    //     }
    // };


    // The main function that handles the slab update action
    const handleSlabUpdate = async () => {
        try {
            if (!selectedItemForVolume || !selectedItemForVolume.id) {
                setError("No valid item selected for update");
                return;
            }

            // Check if rate is below minimum
            const isBelowMinimum = checkRateBelowMinimum();

            // If below minimum, ask for confirmation
            if (isBelowMinimum) {
                setDialog({
                    open: true,
                    type: DialogType.CONFIRM,
                    action: () => performSlabUpdate(),
                    msg: `${rateBelowMinimumWarning} Are you sure you want to continue with a rate below the minimum?`,
                    title: "Confirm Rate Below Minimum",
                });
                return;
            }

            // If not below minimum, proceed with update
            await performSlabUpdate();

        } catch (error) {
            console.error('Error in handleSlabUpdate:', error);
            setError('Failed to process the update request');
        }
    };

    // Separate function that handles the actual API call and update process
    const performSlabUpdate = async () => {
        setError(null);
        dispatch(addLoader("UPDATE_SLAB"));

        try {
            // Find the original item in the proposal items to get the fee_rack_rate_id
            const originalItem = proposal.items?.find(
                item => item.id === selectedItemForVolume.id || item.item_id === selectedItemForVolume.id
            );

            if (!originalItem || !originalItem.fee_rack_rate_id) {
                console.error("Cannot find fee_rack_rate_id for the selected item");
                setError("Missing required data for update. Please try again.");
                dispatch(removeLoader("UPDATE_SLAB"));
                return;
            }

            console.log("Current selected item for volume:", selectedItemForVolume);

            // Prepare the payload with all necessary fields from the controller
            const updatedItemData = {
                item_id: selectedItemForVolume.id || selectedItemForVolume.item_id,
                proposal_id: proposalId,
                fee_rack_rate_id: originalItem.fee_rack_rate_id,
                fee: selectedItemForVolume.fee_type === 'FLAT' ? parseFloat(selectedItemForVolume.fee_flat) : 0,
                fee_percentage: selectedItemForVolume.fee_type === 'PERCENTAGE' || selectedItemForVolume.fee_type === 'MIXED'
                    ? parseFloat(selectedItemForVolume.fee_percentage)
                    : 0,
                negotiation_level: originalItem.negotiation_level || 0,
                item_version: originalItem.item_version || 1.0,
                negotiation_count: originalItem.negotiation_count || 0,
                min_value: originalItem.min_value || originalItem.slab_from || 0,
                max_value: originalItem.max_value || originalItem.slab_to || null,
                velocity_type: selectedItemForVolume.volume_type === 'AMOUNT' ? 1 : 2,
                velocity_period: originalItem.velocity_period || 2,
                userId: JSON.parse(localStorage.getItem('user'))?.username || 'system',
                commited_volume: selectedItemForVolume.committed_volume
                    ? parseFloat(selectedItemForVolume.committed_volume)
                    : 0
            };

            console.log("Updating item with data:", updatedItemData);

            const response = await request({
                api: updateItemVolumeAPI,
                params: { id: selectedItemForVolume.id || selectedItemForVolume.item_id },
                payload: updatedItemData
            });

            if (response.status === 200) {
                renderToaster({
                    type: "success",
                    message: "Volume commitment and rate updated successfully"
                });

                // If the updated rate is below minimum and proposal is approved,
                // we might need to change the proposal status to pending approval
                if (rateBelowMinimumWarning && proposal.proposal_status === 'APPROVED') {
                    try {
                        await request({
                            api: updateProposalStatusAPI,
                            payload: {
                                proposal_id: Number(proposalId),
                                new_status: "PENDING_APPROVAL",
                                comments: "Automatically changed to pending approval due to rate below minimum"
                            },
                        });
                        console.log("Updated proposal status to PENDING_APPROVAL due to rate below minimum");
                    } catch (statusError) {
                        console.error("Error updating proposal status:", statusError);
                    }
                }

                await getProposalById();
                setEditVolumeModal(false);
                setRateBelowMinimumWarning('');
                setSelectedItemForVolume({
                    id: null,
                    country_code: '',
                    payout_type: '',
                    payout_method: '',
                    committed_volume: 0,
                    fee_type: 'FLAT',
                    fee_flat: 0,
                    fee_percentage: 0,
                    volume_type: 'AMOUNT'
                });
            } else {
                setError((response.data && response.data.message) || 'Failed to update volume commitment and rate');
            }
        } catch (error) {
            console.error('Error updating volume and rate:', error);
            setError(error.response?.data?.message || 'Failed to update volume commitment and rate');
        } finally {
            dispatch(removeLoader("UPDATE_SLAB"));
        }
    };

    // const getStatusTextColor = (status) => {
    //     switch (status) {
    //         case "PENDING_APPROVAL":
    //             return Colors.tpPending; // Orange
    //         case "APPROVED":
    //             return Colors.tpSuccess; // Green
    //         case "REJECTED":
    //             return Colors.tpRejectColor; // Red
    //         case "RECALLED":
    //             return Colors.tpTextLight; // Grey
    //         case "EXPIRED":
    //             return Colors.tpTextLight; // Grey
    //         case "SIGNED_OFF":
    //             return Colors.tpBlue; // Blue
    //         default:
    //             return Colors.tpTextLight;
    //     }
    // };

    const getLatestChangedHistory = useMemo(() => {
        const status_history = proposal?.proposal_histories
            ?.filter((p) => p.change_type == "STATUS_CHANGED")
            ?.sort((a, b) => new Date(b.changed_at) - new Date(a.changed_at));
        const status = status_history?.[0] || {};
        const user = userList?.find((u) => u.account_id == status?.changed_by);
        status.changedBy = user;
        return status;
    }, [proposal, userList]);

    const getUserName = (userId) => {
        return userList?.find((u) => u.account_id == userId)?.username;
    };

    const updateProposalData = async (status) => {
        dispatch(addLoader("UPDATE"));
        const { partner: _ignore_p, ...data } = proposal;
        try {
            const response = await request({
                api: updateProposalAPI,
                payload: {
                    ...data,
                    proposal_status: status || data.proposal_status,
                    expiry_date: new Date(
                        proposal?.new_expiry_date || proposal.expiry_date
                    )?.toISOString(),
                },
                params: { id: proposalId },
            });
            if (["200", "201"].includes(response?.status?.toString())) {
                await getProposalById();
                renderToaster({
                    type: "success",
                    message: "Proposal updated successfully",
                });
            }
        } catch (error) {
            setError("Failed to update proposal");
            console.error("Update error:", error);
        } finally {
            setDialogState({
                open: false,
                type: null,
                proposalId: null,
            });
            dispatch(removeLoader("UPDATE"));
        }
    };

    useEffect(() => {
        const getSpeedTextAPICall = async () => {
            dispatch(addLoader("SPEED_TEXT"));
            try {
                const response = await request({
                    api: getFeeRackSpeedTexts,
                });
                if (response.status === 200) {
                    const paymentInstruments = response.data.data2?.map((item) => ({
                        label: item.type,
                        value: item.payment_instrument_id,
                        name: item.payment_instrument_name,
                        type: item.type?.toUpperCase(),
                    }));
                    setPaymentInstruments(paymentInstruments);
                }
            } catch (error) {
                console.error("Error fetching speed texts:", error);
            } finally {
                dispatch(removeLoader("SPEED_TEXT"));
            }
        };

        if (dispatch) getSpeedTextAPICall();
    }, [dispatch]);

    useEffect(() => {
        const getFeeRacks = async (countries) => {
            try {
                const response = await request({
                    api: getFeeRackRate,
                    params: `&country=${countries}`,
                });
                if (response?.status == 200) {
                    setFeeRack(response?.data?.data);
                    const countryWiseRacks = {};
                    const data = response?.data?.data;
                    // setRackRate(data);
                    data?.forEach((fee_rack) => {
                        countryWiseRacks[fee_rack.country_code] = [
                            ...(countryWiseRacks[fee_rack.country_code] || []),
                            fee_rack,
                        ];
                    });

                    console.log("COUNTRY WISE RACKS:", countryWiseRacks);

                    setCountryWiseData(countryWiseRacks);
                }
            } catch (error) {
                console.error("Error fetching fee racks:", error);
            }
        };
        if (proposal.country_codes?.length) {
            getFeeRacks(proposal.country_codes?.join(","));
        }
    }, [proposal?.country_codes]);

    useEffect(() => {
        if (proposalId) {
            getProposalById();
        }
    }, [proposalId, getProposalById]);

    useEffect(() => {
        checkRateBelowMinimum();
    }, [selectedItemForVolume]);

    useEffect(() => {
        filterSlabsForEdit(availableVolumeSlabs, selectedItemForVolume.committed_volume, selectedItemForVolume.volume_type);
    }, [availableVolumeSlabs, selectedItemForVolume.committed_volume, selectedItemForVolume.volume_type, selectedItemForVolume.fee_type]);


    const handleStatusUpdate = async (comments) => {
        try {
            setError("");

            if (dialogState.type === "sign_off") {
                const response = await request({
                    api: updateProposalStatusAPI,
                    payload: {
                        proposal_id: dialogState.proposalId,
                        new_status: "SIGNED_OFF",
                        comments: comments || "", // Allow empty comments
                    },
                });
                // Reset dialog state and refresh proposals
                setDialogState({
                    open: false,
                    type: null,
                    proposalId: null,
                });
                getProposalById();
                return;
            }

            // Rest of your existing status update logic
            if (dialogState.type === "move_to_pending") {
                const response = await request({
                    api: updateProposalStatusAPI,
                    payload: {
                        proposal_id: dialogState.proposalId,
                        comments: comments,
                        new_status: "PENDING_APPROVAL",
                    },
                });
            } else if (dialogState.type === "finalize") {
                const response = await request({
                    api: updateProposalStatusAPI,
                    payload: {
                        proposal_id: dialogState.proposalId,
                        comments: comments,
                        new_status: "FINALIZED",
                    },
                });
            } else if (dialogState.type === "recall") {
                const response = await request({
                    api: updateProposalStatusAPI,
                    payload: {
                        proposal_id: dialogState.proposalId,
                        comments: comments,
                        new_status: "RECALLED",
                    },
                });
            } else {
                const status = dialogState.type === "approve" ? "APPROVED" : "REJECTED";
                const response = await request({
                    api: updateProposalStatusAPI,
                    payload: {
                        proposal_id: Number(dialogState.proposalId),
                        new_status: status,
                        comments: comments || "",
                    },
                });
            }

            setDialogState({
                open: false,
                type: null,
                proposalId: null,
            });
            getProposalById();
        } catch (error) {
            console.error("Error updating proposal status:", error);

            // Check if this is a status validation error (race condition)
            if (error.response?.status === 400 && error.response?.data?.error?.includes("Cannot change proposal status")) {
                // This is likely a race condition where another user has already changed the status
                // Refresh the proposal to get the latest status
                getProposalById();

                setError(
                    "This proposal's status has already been changed by another user. The latest status is now shown."
                );

                // Close the dialog
                setDialogState({
                    open: false,
                    type: null,
                    proposalId: null,
                });

                return;
            }

            // For other errors, show the standard error message
            setError(
                error.response?.data?.error || "Failed to update proposal status"
            );
        }
    };

    const StatusUpdateDialog = () => {
        return (
            <CustomModal
                isOpen={dialogState.open}
                onClose={() =>
                    setDialogState({
                        open: false,
                        type: null,
                        proposalId: null,
                    })
                }
                onSubmit={dialogState.action ? dialogState.action : handleStatusUpdate}
                title={
                    dialogState.type === "approve"
                        ? "Approve Proposal"
                        : dialogState.type === "reject"
                            ? "Reject Proposal"
                            : dialogState.type === "revoke"
                                ? "Revoke Proposal"
                                : dialogState.type === "move_to_pending"
                                    ? "Submit Proposal"
                                    : dialogState.type === "sign_off"
                                        ? "Sign Off Proposal"
                                        : dialogState.type === "finalize"
                                            ? "Finalize Proposal"
                                            : "Update Proposal"
                }
                actionButtonText={
                    dialogState.type === "approve"
                        ? "Approve"
                        : dialogState.type === "reject"
                            ? "Reject"
                            : dialogState.type === "revoke"
                                ? "Revoke"
                                : dialogState.type === "move_to_pending"
                                    ? "Submit"
                                    : dialogState.type === "sign_off"
                                        ? "Sign Off"
                                        : dialogState.type === "finalize"
                                            ? "Finalize Proposal"
                                            : "Update"
                }
                isApprove={dialogState.type === "approve"}
                proposalId={dialogState.proposalId}
            />
        );
    };

    const getVelocityTypes = ({ country_code, payout_currency, payout_type, coverage }) => {
        const data = countryWiseData[country_code]?.filter(
            (frr) =>
                frr.payout_currency == payout_currency && frr.payout_type == payout_type && frr.coverage == coverage
        );
        if (!data) return [];
        const values = [];
        data.forEach((c) => {
            if (!values.find((v) => v == (c.velocity_type || 2))) {
                values.push(c.velocity_type || 2);
            }
        });
        return values;
    };

    const getVelocityPeriods = ({
        country_code,
        payout_currency,
        payout_type,
        coverage,
    }) => {
        const data = countryWiseData[country_code]?.filter(
            (frr) =>
                frr.payout_currency == payout_currency && frr.payout_type == payout_type && frr.coverage == coverage
        );
        if (!data) return [];
        const values = [];

        data.forEach((c) => {
            if (!values.find((v) => v == (c.velocity_period || 2))) {
                values.push(c.velocity_period || 2);
            }
        });
        return values;
    };

    const getRanges = (row) => {
        const { country_code, payout_currency, payout_type, coverage } = row;
        const data = countryWiseData?.[country_code]?.filter(
            (frr) =>
                frr.payout_currency == payout_currency && frr.payout_type == payout_type && frr.coverage == coverage
        );
        if (!data) return [];
        const t_options = getVelocityTypes(row);
        const p_options = getVelocityPeriods(row);
        const ranges = [];
        const default_t = t_options?.[0]?.value || 2;
        const default_p = p_options?.[0]?.value || 2;

        data.forEach((d) => {
            if (
                (d.velocity_period || default_p) ==
                (row.velocity_period || default_p) &&
                (d.velocity_type || default_t) == (row.velocity_type || default_t) &&
                (!row.payout_type || d.payout_type == row.payout_type)
            )
                ranges.push({
                    min_value: d.min_value || 0,
                    max_value: d.max_value || 999999999,
                    id: d.frr_id,
                    otherData: d,
                });
        });

        console.log("Range", data, ranges);
        return ranges;
    };

    const getCalcModes = ({ country_code, payout_currency, payout_type, coverage }) => {
        const data = countryWiseData?.[country_code]?.filter(
            (frr) =>
                frr.payout_currency == payout_currency && frr.payout_type == payout_type && frr.coverage == coverage
        );
        if (!data) return [];
        const values = [];
        data.forEach((d) => {
            if (!values.find((v) => v.value == d.calculation_mode)) {
                values.push({
                    label: d.calculation_mode,
                    value: d.calculation_mode,
                });
            }
        });
        return values;
    };

    const getCoverageOptions = ({ country_code, payout_type, payout_currency }) => {
        const coverageOptions = countryWiseData[country_code]?.filter(
            (item) => item.payout_type === payout_type && item.payout_currency === payout_currency
        );

        const values = [];

        coverageOptions?.forEach((item) => {
            if (!values.some(value => value.coverage === item.coverage)) {
                values.push({ coverage: item.coverage, frr_id: item.id || item.frr_id });
            }
        });

        return values;
    }

    const getTransactionTypeOptions = ({ country_code, payout_type, payout_currency, coverage }) => {
        const transactionTypeOptions = countryWiseData[country_code]?.filter(
            (item) => item.payout_type === payout_type && item.payout_currency === payout_currency && item.coverage === coverage
        );
        const values = [];

        transactionTypeOptions?.forEach((item) => {
            if (!values.some(value => value.transaction_type === item.transaction_type)) {
                values.push({ transaction_type: item.transaction_type, frr_id: item.id || item.frr_id });
            }
        });

        return values;
    }

    const fetchAvailableSlabs = async (countryCode, payoutType) => {
        try {
            dispatch(addLoader("SLABS"));
            console.log(`Fetching slabs for country ${countryCode} and payout_type ${payoutType}`);

            const response = await request({
                api: getFeeRackRate,
                params: `&country=${countryCode}&payout_type=${payoutType}`,
            });

            if (response.status === 200) {
                // Further filter the results to ensure we only get slabs for this payout_type
                const filteredSlabs = response.data.data.filter(slab =>
                    String(slab.payout_type) === String(payoutType)
                );

                setAvailableVolumeSlabs(filteredSlabs);
                console.log(`Found ${filteredSlabs.length} slabs for country ${countryCode} and payout_type ${payoutType}`);

                // If there's already a committed volume, filter slabs right away
                if (selectedItemForVolume.committed_volume) {
                    // filterSlabsForEdit(filteredSlabs, selectedItemForVolume.committed_volume, selectedItemForVolume.volume_type);
                } else {
                    // Clear filtered slabs if no volume
                    // setFilteredEditSlabs([]);
                }
            } else {
                setAvailableVolumeSlabs([]);
                // setFilteredEditSlabs([]);
                setError('No slabs found for this country/payout combination');
            }
        } catch (error) {
            console.error('Error fetching volume slabs:', error);
            setError('Failed to fetch volume slabs');
            setAvailableVolumeSlabs([]);
            // setFilteredEditSlabs([]);
        } finally {
            dispatch(removeLoader("SLABS"));
        }
    };

    const formatFeeDisplay = (feeType, feeFlat = 0, feePercentage = 0) => {
        console.log("Formatting fee display:", { feeType, feeFlat, feePercentage });

        // First ensure we have valid numbers
        const flatFee = typeof feeFlat === 'number' ? feeFlat : parseFloat(feeFlat || 0);
        const percentFee = typeof feePercentage === 'number' ? feePercentage : parseFloat(feePercentage || 0);

        // Avoid NaN by using isNaN check
        const formattedFlat = isNaN(flatFee) ? "0.00" : flatFee.toFixed(2);
        const formattedPercent = isNaN(percentFee) ? "0.00" : percentFee.toFixed(2);

        // Normalize fee type to uppercase
        const normalizedFeeType = (feeType || 'FLAT').toUpperCase();

        switch (normalizedFeeType) {
            case 'FLAT':
                return `${formattedFlat} USD`;
            case 'PERCENTAGE':
                return `${formattedPercent}%`;
            case 'MIXED':
                return `${formattedFlat} USD + ${formattedPercent}%`;
            default:
                return '0.00 USD';
        }
    };

    // Helper function to format mixed fee for enhanced display
    const formatMixedFeeEnhanced = (flatValue, percentValue) => {
        const formattedFlat = isNaN(parseFloat(flatValue)) ? "0" : parseFloat(flatValue).toFixed(0);
        const formattedPercent = isNaN(parseFloat(percentValue)) ? "0" : parseFloat(percentValue).toFixed(0);
        return `${formattedFlat} USD + ${formattedPercent}% of txn value`;
    };

    // Helper function to get available volume types for selected country + payout method
    const getAvailableVolumeTypes = () => {
        if (!newItem.country_code || !newItem.payout_method) {
            return ['AMOUNT', 'TRANSACTION_COUNT']; // Show all if nothing selected
        }

        // Filter fee rack rates for the selected country and payout method
        const relevantRates = feeRack.filter(rate =>
            rate.country_code === newItem.country_code &&
            String(rate.payout_type) === String(newItem.payout_method)
        );

        // Extract unique volume types from the relevant rates
        const uniqueVolumeTypes = [...new Set(relevantRates.map(rate => {
            // Map velocity_type to volume type names
            if (rate.velocity_type === 1) return 'TRANSACTION_COUNT';
            if (rate.velocity_type === 2) return 'AMOUNT';
            return 'AMOUNT'; // Default
        }))];

        return uniqueVolumeTypes.length > 0 ? uniqueVolumeTypes : ['AMOUNT']; // Default to AMOUNT if none found
    };

    // Helper function to get velocity type label
    const getVelocityTypeLabel = (velocityType = 2) => {
        const types = {
            0: 'No Range',
            1: 'Transaction Count Based',
            2: 'Amount Based'
        };
        return types[velocityType] || 'Unknown';
    };

    // Helper function to get velocity period label  
    const getVelocityPeriodLabel = (velocityPeriod = 2) => {
        const periods = {
            0: 'Daily',
            1: 'Weekly',
            2: 'Monthly',
            3: 'Yearly'
        };
        return periods[velocityPeriod] || 'Unknown';
    };


    // Helper function to format negotiation levels
    const formatNegotiationLevels = (slab) => {
        const levels = [];

        // For mixed rates, handle both regular and custom negotiation levels
        if (slab.custom_fee_enable === 1) {
            // Mixed rate - show both components for each level
            if (slab.proposed_fee) {
                levels.push({
                    level: 'Proposed',
                    value: formatMixedRateLevel(slab, 'proposed_fee', 'custom_proposed_fee')
                });
            }

            if (slab.fee_negotiation_level_1 || slab.custom_fee_negotiation_level_1) {
                levels.push({
                    level: 'Level 1',
                    value: formatMixedRateLevel(slab, 'fee_negotiation_level_1', 'custom_fee_negotiation_level_1')
                });
            }

            if (slab.fee_negotiation_level_2 || slab.custom_fee_negotiation_level_2) {
                levels.push({
                    level: 'Level 2',
                    value: formatMixedRateLevel(slab, 'fee_negotiation_level_2', 'custom_fee_negotiation_level_2')
                });
            }

            if (slab.minimum_fee || slab.custom_minimum_fee) {
                levels.push({
                    level: 'Minimum/Walkaway',
                    value: formatMixedRateLevel(slab, 'minimum_fee', 'custom_minimum_fee')
                });
            }
        } else {
            // Single rate
            if (slab.proposed_fee) {
                levels.push({
                    level: 'Proposed',
                    value: slab.calculation_mode === 'percentage' ? `${slab.proposed_fee}%` : `${slab.proposed_fee} USD`
                });
            }

            if (slab.fee_negotiation_level_1) {
                levels.push({
                    level: 'Level 1',
                    value: slab.calculation_mode === 'percentage' ? `${slab.fee_negotiation_level_1}%` : `${slab.fee_negotiation_level_1} USD`
                });
            }

            if (slab.fee_negotiation_level_2) {
                levels.push({
                    level: 'Level 2',
                    value: slab.calculation_mode === 'percentage' ? `${slab.fee_negotiation_level_2}%` : `${slab.fee_negotiation_level_2} USD`
                });
            }

            if (slab.minimum_fee) {
                levels.push({
                    level: 'Minimum/Walkaway',
                    value: slab.calculation_mode === 'percentage' ? `${slab.minimum_fee}%` : `${slab.minimum_fee} USD`
                });
            }
        }

        return levels;
    };

    // Helper function to format mixed rate levels
    const formatMixedRateLevel = (slab, primaryField, customField) => {
        if (!slab) return 'Not configured';

        const primaryValue = slab[primaryField];
        let secondaryValue = null;

        if (slab[customField]) {
            try {
                const customData = JSON.parse(slab[customField]);
                secondaryValue = customData?.fees;
            } catch (e) {
                console.log(`Failed to parse ${customField} JSON`, e);
            }
        }

        if (primaryValue && secondaryValue) {
            const calcMode = slab.calculation_mode || 'flat';
            const primaryFormatted = calcMode === 'percentage' ? `${primaryValue}%` : `${primaryValue} USD`;
            const secondaryFormatted = calcMode === 'percentage' ? `${secondaryValue} USD` : `${secondaryValue}%`;
            return `${primaryFormatted} + ${secondaryFormatted}`;
        } else if (primaryValue) {
            const calcMode = slab.calculation_mode || 'flat';
            return calcMode === 'percentage' ? `${primaryValue}%` : `${primaryValue} USD`;
        }

        return 'Not configured';
    };

    // Helper function to format mixed fee information
    const formatMixedFeeInfo = (slab) => {
        if (!slab || slab.custom_fee_enable !== 1) return null;

        try {
            const customFee = slab.custom_proposed_fee ? JSON.parse(slab.custom_proposed_fee) : null;
            if (customFee && customFee.fees) {
                return {
                    primary: {
                        type: slab.calculation_mode || 'flat',
                        value: slab.proposed_fee || '0'
                    },
                    secondary: {
                        type: customFee.calculationMode || 'percentage',
                        value: customFee.fees || '0'
                    }
                };
            }
        } catch (e) {
            console.log("Failed to parse custom fee JSON", e);
        }

        return null;
    };


    const handleFeeValueApplyToAll = useCallback((feeValue, index, check, type) => {
        if (!feeValue?.value) {
            return;
        }
        // Make a copy to avoid mutating the original
        const updatedData = proposal?.items?.map((item, idx) => {
            // Skip processing if this item has been manually edited by the user
            if (check && index !== undefined && index !== idx) {
                console.log(`Skipping auto-processing for user-edited item: ${item.country_code}`);
                return item;
            }
            if (check == undefined && !item.check_box) {
                console.log(`Skipping auto-processing for user-edited item: ${item.country_code}`);
                return item;
            }
            item.userEdited = true;

            const fee = item[feeValue.value] || item?.fee_rack_rate?.[feeValue.value];
            const fee2 = item[feeValue.value2] || item?.fee_rack_rate?.[feeValue.value2];

            // Handle mixed rates with custom_fee_enable
            if (item.custom_fee_enable === 1 || item.fee_type === "MIXED") {
                console.log(`Processing mixed rate for ${item.country_code}, calculation_mode: ${item.calculation_mode}`);
                const feeData = fee;
                const fee2Data = fee2;

                if (item.calculation_mode === "percentage") {
                    // For percentage primary mode, secondary is flat
                    const percentageValue = feeData || 0;
                    const flatValue = (() => {
                        try {
                            // Try to parse the custom_proposed_fee as JSON
                            const customFee = fee2Data ? JSON.parse(fee2Data) : null;
                            // If parsing succeeded and has a fees property, use it
                            if (customFee && customFee.fees) {
                                return parseFloat(customFee.fees);
                            }
                        } catch (e) {
                            // If JSON parsing fails, fall back to direct field value
                            console.log("Failed to parse custom fee JSON", e);
                        }
                        return parseFloat(item.fee_flat || 0);
                    })();

                    const updatedItem = {
                        ...item,
                        fee_type: "MIXED",
                        fee_percentage: index != undefined ? type === 'percentage' ? percentageValue : item.fee_percentage : percentageValue,
                        fee: index != undefined ? type === 'flat' ? flatValue : item.fee : percentageValue, // Set fee equal to fee_percentage for percentage mode
                        fee_flat: index != undefined ? type === 'flat' ? flatValue : item.fee_flat : flatValue, // Explicitly set fee_flat
                    };
                    console.log(`Updated percentage-primary mixed rate: flat=${updatedItem.fee_flat}, percentage=${updatedItem.fee_percentage}`);
                    return updatedItem;
                } else {
                    // For flat primary mode, secondary is percentage
                    const flatValue = feeData || 0;
                    const percentageValue = (() => {
                        try {
                            // Try to parse the custom_proposed_fee as JSON
                            const customFee = fee2Data ? JSON.parse(fee2Data) : null;
                            // If parsing succeeded and has a fees property, use it
                            if (customFee && customFee.fees) {
                                return parseFloat(customFee.fees);
                            }
                        } catch (e) {
                            // If JSON parsing fails, fall back to direct field value
                            console.log("Failed to parse custom fee JSON", e);
                        }
                        return parseFloat(item.fee_percentage || 0);
                    })();

                    const updatedItem = {
                        ...item,
                        fee_type: "MIXED",
                        fee: index != undefined ? type === 'flat' ? flatValue : item.fee : flatValue,
                        fee_flat: index != undefined ? type === 'flat' ? flatValue : item.fee_flat : flatValue, // Explicitly set fee_flat
                        fee_percentage: index != undefined ? type === 'percentage' ? percentageValue : item.fee_percentage : percentageValue, // Explicitly set fee_percentage
                    };
                    console.log(`Updated flat-primary mixed rate: flat=${updatedItem.fee_flat}, percentage=${updatedItem.fee_percentage}`);
                    return updatedItem;
                }
            }
            // Handle standard percentage-only rates
            else if (item.calculation_mode === "percentage") {
                return {
                    ...item,
                    fee_type: "PERCENTAGE",
                    fee_percentage: index != undefined ? type === 'percentage' ? fee : item.fee_percentage : (fee || 0),
                    fee: index != undefined ? type === 'flat' ? item.fee_flat : item.fee : (fee || 0), // Set fee equal to fee_percentage for percentage mode
                };
            }
            // Handle standard flat-only rates
            else {
                return {
                    ...item,
                    fee_type: "FLAT",
                    fee: index != undefined ? type === 'flat' ? fee : item.fee : (fee || 0),
                    fee_percentage: null,
                };
            }
        });

        // Only update if there's a difference to avoid infinite loops
        if (JSON.stringify(updatedData) !== JSON.stringify(proposal?.items)) {
            console.log("Updating pricing data with processed mixed rates");
            setProposal((prev) => ({
                ...prev,
                items: updatedData,
            }));
        }
    }, [proposal?.items]);

    // Add a loading indicator
    if (proposal.loading) {
        return (
            <Box sx={{ width: "100%", height: "100%", display: "flex", justifyContent: "center", alignItems: "center" }}>
                <Typography>Loading proposal data...</Typography>
            </Box>
        );
    }

    return (
        <Box sx={{ width: "100%", height: "100%" }}>
            <button
                type="button"
                className="btn add-btn d-flex p-0 p-1"
                onClick={() => navigate(-1)}
            >
                <i className="bx bx-chevron-left" style={{ marginRight: "4px" }} />
                Back
            </button>
            <Card
                variant="outlined"
                sx={{ w: "100%", h: "100%", px: 2, py: 1, mt: 1 }}
            >
                <Stack direction="row" justifyContent="space-between">
                    <Stack direction="row" gap={3}>
                        <TextMedium fontSize="22px" fontWeight="100">
                            Proposal: {proposal?.proposal_name}
                        </TextMedium>
                        <TextMedium fontSize="22px" fontWeight="100">
                            Version: {proposal?.ver}
                        </TextMedium>
                    </Stack>
                </Stack>

                {error && (
                    <Alert
                        severity="error"
                        sx={{ mb: 3 }}
                        onClose={() => {
                            setError("");
                        }}
                    >
                        {error}
                    </Alert>
                )}
                <Box mt={2} width="100%">
                    <Grid2 container spacing={2} alignItems="center">
                        <Grid2
                            item
                            size={{
                                xl: 3,
                                md: 4,
                                sm: 6,
                                xs: 12,
                            }}
                            container
                        >
                            <Grid2 item size={4}>
                                <TextMedium>Partner Name:</TextMedium>
                            </Grid2>
                            <Grid2 item size={8}>
                                <TextMedium>{proposal?.partner?.partner_name}</TextMedium>
                            </Grid2>
                        </Grid2>
                        <Grid2
                            item
                            size={{
                                xl: 3,
                                md: 4,
                                sm: 6,
                                xs: 12,
                            }}
                            container
                        >
                            <Grid2 item size={4}>
                                <TextMedium>Status:</TextMedium>
                            </Grid2>
                            <Grid2 item size={8}>
                                <TextNormal>
                                    <Box
                                        sx={{
                                            display: "inline-flex",
                                            alignItems: "center",
                                            justifyContent: "flex-start",
                                            borderRadius: "4px",
                                            fontWeight: 500,
                                            fontSize: "0.875rem",
                                            fontFamily: "Space Grotesk",
                                            textAlign: "left", // Changed from center to left
                                            padding: 0, // Set all padding to 0
                                            margin: 0 // Set all margin to 0
                                        }}
                                    >
                                        {proposal?.proposal_status === "SIGNED_OFF"
                                            ? "Signed Off"
                                            : proposal?.proposal_status === "PENDING_APPROVAL"
                                                ? "Pending Approval"
                                                : proposal?.proposal_status === "APPROVED"
                                                    ? "Approved"
                                                    : proposal?.proposal_status === "REJECTED"
                                                        ? "Rejected"
                                                        : proposal?.proposal_status === "RECALLED"
                                                            ? "Recall"
                                                            : proposal?.proposal_status === "EXPIRED"
                                                                ? "Expired"
                                                                : proposal?.proposal_status}
                                    </Box>
                                </TextNormal>
                            </Grid2>
                        </Grid2>
                        {/* {["APPROVED", "REJECTED", "RECALLED"].includes(
                            proposal.proposal_status
                        ) && (
                                <Grid2
                                    item
                                    size={{
                                        xl: 3,
                                        md: 4,
                                        sm: 6,
                                        xs: 12,
                                    }}
                                    container
                                >
                                    <Grid2 item size={4}>
                                        <TextMedium variant="subtitle2">
                                            {proposal.proposal_status === "APPROVED" && "Approved by:"}
                                            {proposal.proposal_status === "REJECTED" && "Rejected by:"}
                                            {proposal.proposal_status === "RECALLED" && "Recalled by:"}
                                        </TextMedium>
                                    </Grid2>
                                    <Grid2 item size={8}>
                                        <TextNormal>
                                            { proposal.approved_by === "system" ? "System" : (getUserName(proposal.approved_by) || "Not available") }
                                        </TextNormal>
                                    </Grid2>
                                </Grid2>
                            )} */}
                        <Grid2
                            item
                            size={{
                                xl: 3,
                                md: 4,
                                sm: 6,
                                xs: 12,
                            }}
                            container
                        >
                            <Grid2 item size={4}>
                                <TextMedium>Created By:</TextMedium>
                            </Grid2>
                            <Grid2 item size={8}>
                                <TextNormal>
                                    {getUserName(proposal.created_by) || "User not found"}
                                </TextNormal>
                            </Grid2>
                        </Grid2>
                        <Grid2
                            item
                            size={{
                                xl: 3,
                                md: 4,
                                sm: 6,
                                xs: 12,
                            }}
                            container
                            alignItems="center"
                        >
                            <Grid2 item size={4}>
                                <TextMedium>Expiry Date:</TextMedium>
                            </Grid2>
                            <Grid2 item size={8}>
                                <BasicDatePicker
                                    selectedDate={new Date(proposal.new_expiry_date || proposal.expiry_date)}
                                    onChange={handleExpiryDateChange}
                                    minDate={new Date(new Date().setDate(new Date().getDate() + 1))} // Tomorrow
                                    shouldDisableDate={(date) => {
                                        const today = new Date();
                                        today.setHours(0, 0, 0, 0);
                                        return date <= today;
                                    }}
                                    helperText={
                                        !isValidExpiryDate(proposal.expiry_date)
                                            ? "Expiry date must be in the future"
                                            : ""
                                    }
                                    error={!isValidExpiryDate(proposal.expiry_date)}
                                />
                            </Grid2>
                        </Grid2>
                    </Grid2>
                </Box>
                <Stack sx={{ gap: 2, my: 2 }}>
                    <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', mb: 1 }}>
                        <TextBoldBig>Proposal Items</TextBoldBig>

                        {access.EDIT && (
                            // proposal.proposal_status === 'PENDING_APPROVAL' ||
                            proposal.proposal_status === 'RECALLED' ||
                            proposal.proposal_status === 'APPROVED' ||
                            proposal.proposal_status === 'REJECTED') && (
                                <Button
                                    variant="contained"
                                    color="primary"
                                    startIcon={<Add />}
                                    onClick={() => {
                                        console.log("Add country clicked");
                                        setAddCountryModal(true);
                                        fetchCountriesAndMethods();
                                    }}
                                >
                                    Add Country
                                </Button>
                            )}
                        {proposal.proposal_status === 'SENT_TO_PARTNER' &&
                            <Button
                                variant="contained"
                                color="primary"
                                onClick={() => {
                                    setDialogState({
                                        open: true,
                                        type: "finalize",
                                        proposalId: proposal.id,
                                        comments: "",
                                    });
                                }}
                            > Finalize Proposal</Button>
                        }
                    </Box>
                    {proposal?.items?.length > 0 ? (
                        <ItemsTables
                            data={proposal?.items}
                            readOnly={!edit}
                            showActions={(
                                // proposal.proposal_status === 'PENDING_APPROVAL' ||
                                proposal.proposal_status === 'APPROVED' ||
                                proposal.proposal_status === 'RECALLED' ||
                                proposal.proposal_status === 'REJECTED'
                            )}
                            proposal_status={proposal?.proposal_status}
                            setData={(action) => {
                                if (typeof action === "function") {
                                    setProposal((prev) => {
                                        const newItems = action(prev.items);
                                        prev.items = [...newItems];
                                        return { ...prev };
                                    });
                                } else {
                                    const proposalData = { ...proposal };
                                    proposalData.items = [...action];
                                    setProposal(proposalData);
                                }
                            }}
                            getCalcModes={getCalcModes}
                            getRanges={getRanges}
                            getVelocityPeriods={getVelocityPeriods}
                            getVelocityTypes={getVelocityTypes}
                            getCoverageOptions={getCoverageOptions}
                            getTransactionTypeOptions={getTransactionTypeOptions}
                            handleFeeValueApplyToAll={handleFeeValueApplyToAll}
                            onEditVolume={(item) => {
                                console.log("Edit volume clicked for item:", item);

                                // Extract fee value for proper display
                                let extractedFee = null;

                                // If fee is a string (like "4.20 USD"), extract the numeric part
                                if (typeof item.fee === 'string') {
                                    if (item.fee.includes('USD')) {
                                        extractedFee = parseFloat(item.fee.replace(/[^0-9.]/g, ''));
                                    } else if (!isNaN(parseFloat(item.fee))) {
                                        extractedFee = parseFloat(item.fee);
                                    }
                                }

                                // Create a clean version of the item with proper defaults
                                const cleanItem = {
                                    id: item.id || item.item_id,
                                    country_code: item.country_code || '',
                                    payout_type: typeof item.payout_type === 'number' ? String(item.payout_type) : (item.payout_type || ''),
                                    payout_method: item.payout_method || '',
                                    committed_volume: item.committed_volume || item.commited_volume || 0,
                                    fee_type: item.fee_type || 'FLAT',
                                    fee_flat: extractedFee || item.fee_flat || 0,
                                    fee_percentage: item.fee_percentage || 0,
                                    volume_type: item.volume_type || 'AMOUNT'
                                };

                                console.log("Prepared item for edit:", cleanItem);

                                // Set the selected item for editing
                                setSelectedItemForVolume(cleanItem);

                                // Use the string version of payout_type if it's a number for the API call
                                const payoutTypeForAPI = typeof item.payout_type === 'number'
                                    ? String(item.payout_type)
                                    : (item.payout_type || item.payout_method);

                                fetchAvailableSlabs(item.country_code, payoutTypeForAPI);

                                // Open the edit modal
                                setEditVolumeModal(true);
                            }}
                            onDeleteItem={handleDeleteItem}
                            deleteButtonTooltip="Delete Item from Proposal"
                            showEditButton={!!access.EDIT && !edit}
                            showDeleteButton={!!access.DELETE}
                        />
                    ) : (
                        <Typography>No proposal items found</Typography>
                    )}
                    <Stack
                        direction="row"
                        alignContent={"center"}
                        justifyContent={"center"}
                        gap={2}
                    >
                        {edit && (
                            <Stack direction="row" gap={2}>
                                <CommonButton
                                    width="100px"
                                    fullWidth={false}
                                    onClick={() => {
                                        setDialogState({
                                            open: true,
                                            type: "move_to_pending",
                                            proposalId: proposal.id,
                                            comments: "",
                                            action: () => {
                                                updateProposalData("PENDING_APPROVAL");
                                                setEdit(() => false);
                                            }
                                        });
                                    }}
                                >
                                    {"Submit"}
                                </CommonButton>

                                <CommonButton
                                    width="150px"
                                    variant={"outlined"}
                                    fullWidth={false}
                                    onClick={() => {
                                        setProposal(() => ({
                                            ...refOriginalProposal.current,
                                            items: [...(refOriginalProposal.current?.items || [])],
                                        }));
                                        setEdit(() => false);
                                    }}
                                >
                                    {"Cancel edit"}
                                </CommonButton>
                            </Stack>
                        )}
                        <CommonButton
                            width="80px"
                            variant="outlined"
                            fullWidth={false}
                            onClick={() => {
                                navigate(-1);
                            }}
                        >
                            Back
                        </CommonButton>
                        {access.APPROVE && proposal.proposal_status === "PENDING_APPROVAL" &&
                            dec_id !== proposal.created_by && (
                                <>
                                    <Button
                                        variant="outlined"
                                        // sx={{ bgcolor: Colors.tpFailed }}
                                        onClick={() => {
                                            setDialogState({
                                                open: true,
                                                type: "reject",
                                                proposalId: proposal.id,
                                                comments: "",
                                            });
                                        }}
                                    >
                                        Reject
                                    </Button>
                                    <Button
                                        variant="contained"
                                        // sx={{ bgcolor: Colors.tpSuccess }}
                                        onClick={() => {
                                            setDialogState({
                                                open: true,
                                                type: "approve",
                                                proposalId: proposal.id,
                                                comments: "",
                                            });
                                        }}
                                    >
                                        Approve
                                    </Button>
                                </>
                            )}

                        {proposal.proposal_status === "PENDING_APPROVAL" &&
                            getUserName(proposal.created_by) ===
                            (JSON.parse(localStorage.getItem('user'))?.username || '') && (
                                <Typography variant="body2" color="textSecondary">
                                    Waiting for approval
                                </Typography>
                            )}

                        {/* <Button
                            variant="contained"
                            fullWidth={false}
                            onClick={updateProposalData}
                            >
                            Configure and Update
                        </Button> */}
                    </Stack>
                </Stack>
            </Card>
            <StatusUpdateDialog />

            <Dialog open={addCountryModal} onClose={() => setAddCountryModal(false)} maxWidth="sm" fullWidth>
                <DialogTitle>Add Country to Proposal</DialogTitle>
                <DialogContent>
                    <FormControl fullWidth sx={{ mt: 2 }}>
                        <InputLabel>Country</InputLabel>
                        <Select
                            value={newItem.country_code || ''}
                            onChange={(e) => handleCountryChange(e.target.value)}
                            label="Country"
                        >
                            {countries.map((country) => (
                                <MenuItem key={country} value={country}>
                                    {getCountryName(country)}
                                </MenuItem>
                            ))}
                        </Select>
                    </FormControl>

                    <FormControl fullWidth sx={{ mt: 2 }}>
                        <InputLabel>Payout Method</InputLabel>
                        <Select
                            value={newItem.payout_method || ''}
                            onChange={(e) => handlePayoutMethodChange(e.target.value)}
                            disabled={!newItem.country_code}
                            label="Payout Method"
                        >
                            {payoutMethods.map(method => (
                                <MenuItem key={method.id} value={method.id}>
                                    {method.name}
                                </MenuItem>
                            ))}
                        </Select>
                    </FormControl>

                    {/* Add Volume Commitment Section */}
                    <Box sx={{ mt: 3, border: '1px solid #e0e0e0', borderRadius: 1, p: 2 }}>
                        <Typography variant="subtitle1" gutterBottom>
                            Monthly Volume Commitment
                        </Typography>

                        <FormControl fullWidth sx={{ mb: 2 }}>
                            <InputLabel>Volume Type</InputLabel>
                            <Select
                                value={selectedItemForVolume.volume_type || 'AMOUNT'}
                                onChange={(e) => handleEditVolumeTypeChange(e.target.value)}
                                label="Volume Type"
                                disabled={!newItem.country_code || !newItem.payout_method}
                            >
                                {getAvailableVolumeTypes().map(volumeType => (
                                    <MenuItem key={volumeType} value={volumeType}>
                                        {volumeType === 'AMOUNT' ? 'Amount Based' : 'Transaction Count'}
                                    </MenuItem>
                                ))}
                            </Select>
                        </FormControl>

                        <TextField
                            fullWidth
                            label={`Volume Commitment`}
                            value={newItem.committed_volume || ''}
                            onChange={(e) => handleVolumeCommitmentChange(e.target.value)}
                            placeholder={`Enter monthly volume commitment`}
                            sx={{ mb: 2 }}
                        />
                    </Box>

                    {filteredSlabs.length > 0 ? (
                        <Box sx={{ mt: 2 }}>
                            <Typography variant="subtitle1" gutterBottom>
                                Available rate for your volume
                            </Typography>
                            <Grid container spacing={2}>
                                {filteredSlabs.map((slab, index) => {
                                    // Find the corresponding fee rack rate data for complete information
                                    const correspondingFeeRackRate = feeRack.find(rate => rate.frr_id === slab.frr_id) || {};

                                    // Merge slab data with fee rack rate data for complete information
                                    const enrichedSlab = {
                                        ...slab,
                                        ...correspondingFeeRackRate,
                                        // Preserve slab-specific processed fields
                                        fee_flat: slab.fee_flat,
                                        fee_percentage: slab.fee_percentage,
                                        fee_type: slab.fee_type
                                    };
                                    console.log("Enriched slab: index", index, enrichedSlab);

                                    const negotiationLevels = formatNegotiationLevels(enrichedSlab);
                                    const mixedFeeInfo = formatMixedFeeInfo(enrichedSlab);

                                    return (
                                        <Grid item xs={12} key={index}>
                                            <Paper
                                                elevation={3}
                                                sx={{
                                                    p: 3,
                                                    backgroundColor: 'white',
                                                    border: '1px solid #e0e0e0',
                                                    borderLeft: '4px solid #4caf50' // Highlight for slab match
                                                }}
                                            >
                                                {/* Header Section */}
                                                <Box sx={{ mb: 2 }}>
                                                    <Grid container spacing={2} alignItems="center">
                                                        <Grid item xs={12} md={8}>
                                                            <Typography variant="h6" color="primary">
                                                                Rate Configuration Details
                                                            </Typography>
                                                        </Grid>
                                                        <Grid item xs={12} md={4}>
                                                            <Chip
                                                                label={enrichedSlab.status || 'ACTIVE'}
                                                                color={enrichedSlab.status === 'ACTIVE' ? 'success' : 'default'}
                                                                size="small"
                                                            />
                                                            {enrichedSlab.custom_fee_enable === 1 && (
                                                                <Chip
                                                                    label="MIXED FEE"
                                                                    color="warning"
                                                                    size="small"
                                                                    sx={{ ml: 1 }}
                                                                />
                                                            )}
                                                        </Grid>
                                                    </Grid>
                                                </Box>

                                                <Divider sx={{ mb: 2 }} />

                                                {/* Basic Information */}
                                                <Grid container spacing={2} sx={{ mb: 2 }}>
                                                    <Grid item xs={12} md={6}>
                                                        <Typography variant="subtitle2" color="textSecondary">Volume Range</Typography>
                                                        <Typography variant="body1">
                                                            {enrichedSlab.min_value || '0.00'} - {enrichedSlab.max_value === 999999999 ? '∞' : (enrichedSlab.max_value || 'Unlimited')}
                                                            {enrichedSlab.velocity_type === 1 ? ' Transactions' : ' USD'}
                                                        </Typography>
                                                    </Grid>
                                                    <Grid item xs={12} md={6}>
                                                        <Typography variant="subtitle2" color="textSecondary">Fee Calculation Mode</Typography>
                                                        <Typography variant="body1" sx={{ textTransform: 'capitalize' }}>
                                                            {enrichedSlab.custom_fee_enable === 1 ? 'Mixed' : (enrichedSlab.calculation_mode || 'Flat')}
                                                        </Typography>
                                                    </Grid>
                                                    <Grid item xs={12} md={6}>
                                                        <Typography variant="subtitle2" color="textSecondary">Payout Currency</Typography>
                                                        <Typography variant="body1">{enrichedSlab.payout_currency || 'USD'}</Typography>
                                                    </Grid>
                                                    <Grid item xs={12} md={6}>
                                                        <Typography variant="subtitle2" color="textSecondary">Payout Method</Typography>
                                                        <Typography variant="body1">{getPayoutMethodName(enrichedSlab.payout_type)}</Typography>
                                                    </Grid>
                                                    <Grid item xs={12} md={6}>
                                                        <Typography variant="subtitle2" color="textSecondary">Coverage</Typography>
                                                        <Typography variant="body1">{enrichedSlab.coverage || 'Standard'}</Typography>
                                                    </Grid>
                                                    <Grid item xs={12} md={6}>
                                                        <Typography variant="subtitle2" color="textSecondary">Speed</Typography>
                                                        <Typography variant="body1">{enrichedSlab.speed || 'Standard'}</Typography>
                                                    </Grid>
                                                    {enrichedSlab.transaction_type && (
                                                        <Grid item xs={12} md={6}>
                                                            <Typography variant="subtitle2" color="textSecondary">Transaction Type</Typography>
                                                            <Typography variant="body1">{enrichedSlab.transaction_type == 1 ? "Transaction count" : "Amount based"}</Typography>
                                                        </Grid>
                                                    )}
                                                </Grid>

                                                {/* Velocity Information */}
                                                <Grid container spacing={2} sx={{ mb: 2 }}>
                                                    <Grid item xs={12} md={6}>
                                                        <Typography variant="subtitle2" color="textSecondary">Velocity Type</Typography>
                                                        <Typography variant="body1">
                                                            {getVelocityTypeLabel(enrichedSlab.velocity_type)}
                                                        </Typography>
                                                    </Grid>
                                                    <Grid item xs={12} md={6}>
                                                        <Typography variant="subtitle2" color="textSecondary">Velocity Period</Typography>
                                                        <Typography variant="body1">
                                                            {getVelocityPeriodLabel(enrichedSlab.velocity_period)}
                                                        </Typography>
                                                    </Grid>
                                                </Grid>

                                                <Divider sx={{ mb: 2 }} />

                                                {/* Fee Structure */}
                                                <Box sx={{ mb: 2 }}>
                                                    <Typography variant="subtitle2" color="textSecondary" sx={{ mb: 1 }}>
                                                        Fee
                                                    </Typography>
                                                    <Paper variant="outlined" sx={{ p: 2, bgcolor: 'grey.50' }}>
                                                        <Typography variant="h6" color="primary">
                                                            {mixedFeeInfo ?
                                                                formatMixedFeeEnhanced(
                                                                    mixedFeeInfo.primary.type === 'flat' ? mixedFeeInfo.primary.value : mixedFeeInfo.secondary.value,
                                                                    mixedFeeInfo.primary.type === 'percentage' ? mixedFeeInfo.primary.value : mixedFeeInfo.secondary.value
                                                                ) :
                                                                enrichedSlab.custom_fee_enable === 1 ?
                                                                    formatMixedFeeEnhanced(
                                                                        enrichedSlab.calculation_mode === 'percentage' ? (mixedFeeInfo?.secondary?.value || 0) : enrichedSlab.proposed_fee,
                                                                        enrichedSlab.calculation_mode === 'percentage' ? enrichedSlab.proposed_fee : (mixedFeeInfo?.secondary?.value || 0)
                                                                    ) :
                                                                    formatFeeDisplay(
                                                                        enrichedSlab.calculation_mode || 'FLAT',
                                                                        enrichedSlab.proposed_fee,
                                                                        enrichedSlab.calculation_mode === 'percentage' ? enrichedSlab.proposed_fee : 0
                                                                    )
                                                            }
                                                        </Typography>
                                                    </Paper>
                                                </Box>

                                                {/* Negotiation Levels - Commented out for cleaner UI, can be re-enabled if needed */}
                                                {/*
                                                {negotiationLevels.length > 0 && (
                                                    <Box sx={{ mb: 2 }}>
                                                        <Typography variant="subtitle2" color="textSecondary" sx={{ mb: 1 }}>
                                                            Available Negotiation Levels
                                                        </Typography>
                                                        <List dense>
                                                            {negotiationLevels.map((level, levelIndex) => (
                                                                <ListItem key={levelIndex} sx={{ py: 0.5 }}>
                                                                    <ListItemText
                                                                        primary={
                                                                            <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
                                                                                <Typography variant="body2" color="textSecondary">
                                                                                    {level.level}:
                                                                                </Typography>
                                                                                <Typography variant="body2" sx={{ fontWeight: 'medium' }}>
                                                                                    {level.value}
                                                                                </Typography>
                                                                            </Box>
                                                                        }
                                                                    />
                                                                </ListItem>
                                                            ))}
                                                        </List>
                                                    </Box>
                                                )}
                                                */}

                                                {/* Action Note */}
                                                <Box sx={{ mt: 2, p: 2, bgcolor: 'info.50', borderRadius: 1 }}>
                                                    <Typography variant="body2" color="info.main">
                                                        💡 Add this country to the proposal to edit rates and access all negotiation levels.
                                                    </Typography>
                                                </Box>
                                            </Paper>
                                        </Grid>
                                    );
                                })}
                            </Grid>
                        </Box>
                    ) : newItem.committed_volume ? (
                        <Alert severity="warning" sx={{ mt: 2 }}>
                            Cannot Add: No matching rate slabs found for the specified volume. Please adjust your volume commitment.
                        </Alert>
                    ) : addCountrySlabs.length > 0 ? (
                        <Alert severity="info" sx={{ mt: 2 }}>
                            Enter a volume commitment to see available rate slabs.
                        </Alert>
                    ) : null}

                    <TextField
                        fullWidth
                        label="Reason for Addition (Optional)"
                        multiline
                        rows={3}
                        sx={{ mt: 2 }}
                        value={newItem.reason || ''}
                        onChange={(e) => setNewItem(prev => ({
                            ...prev,
                            reason: e.target.value
                        }))}
                        placeholder="Make a note to remember why this change was made..."
                    />

                    {duplicateWarning && (
                        <Alert severity="error" sx={{ mt: 2 }}>
                            {duplicateWarning}
                        </Alert>
                    )}
                </DialogContent>
                <DialogActions>
                    <Button
                        onClick={() => {
                            setNewItem(() => ({
                                country_code: '',
                                country_name: '',
                                payout_method: '',
                                proposed_rate: '',
                                reason: '',
                                volume_type: 'AMOUNT',
                            }));
                            setAddCountryModal(false)
                        }}
                    >
                        Cancel
                    </Button>
                    <Button
                        onClick={handleAddItem}
                        disabled={!newItem.country_code || !newItem.payout_method || duplicateWarning || isDuplicateItem}
                        variant="contained"
                    >
                        Add Item
                    </Button>
                </DialogActions>
            </Dialog>

            <Dialog
                open={editVolumeModal}
                onClose={() => {
                    setEditVolumeModal(false);
                    setSelectedItemForVolume({
                        id: null,
                        country_code: '',
                        payout_type: '',
                        payout_method: '',
                        committed_volume: 0,
                        fee_type: 'FLAT',
                        fee_flat: 0,
                        fee_percentage: 0,
                        volume_type: 'AMOUNT'
                    });
                    setHasAttemptedVolumeChange(false); // Reset flag here
                    setError(null);
                }}
                maxWidth="md"
                fullWidth
            >
                <DialogTitle>Edit Volume Commitment or Rate</DialogTitle>
                <DialogContent>
                    <Box sx={{ mt: 2 }}>
                        <FormControl fullWidth sx={{ mb: 2 }}>
                            <InputLabel>Country and Payout Method</InputLabel>
                            <Select
                                value={`${selectedItemForVolume.country_code || ''} - ${selectedItemForVolume.payout_type || selectedItemForVolume.payout_method || ''}`}
                                disabled
                            >
                                <MenuItem value={`${selectedItemForVolume.country_code || ''} - ${selectedItemForVolume.payout_type || selectedItemForVolume.payout_method || ''}`}>
                                    {selectedItemForVolume.country_code ? getCountryName(selectedItemForVolume.country_code) : ''} ({selectedItemForVolume.country_code || ''}) - {
                                        getPayoutMethodName(selectedItemForVolume.payout_type || selectedItemForVolume.payout_method)
                                    }
                                </MenuItem>
                            </Select>
                        </FormControl>
                        <FormControl fullWidth sx={{ mb: 2 }}>
                            <InputLabel>Volume Type</InputLabel>
                            <Select
                                label="Volume Type"
                                value={selectedItemForVolume.volume_type || 'AMOUNT'}
                                onChange={(e) => {
                                    setSelectedItemForVolume({
                                        ...selectedItemForVolume,
                                        volume_type: e.target.value
                                    });
                                }}
                            >
                                {availableVolumeTypes.map((type) => (
                                    <MenuItem key={'volume_type' + type.option} value={type.option}>{type.label}</MenuItem>
                                ))}
                            </Select>
                        </FormControl>

                        <FormControl fullWidth sx={{ mb: 2 }}>
                            <TextField
                                fullWidth
                                label="Monthly Volume Commitment"
                                value={selectedItemForVolume.committed_volume || selectedItemForVolume.commited_volume || ''}
                                onChange={(e) => handleEditVolumeCommitmentChange(e.target.value)}
                                placeholder="Enter monthly volume commitment"
                                InputProps={{
                                    startAdornment: selectedItemForVolume.volume_type === 'AMOUNT' ?
                                        <Box component="span" sx={{ mr: 1 }}>USD</Box> : null,
                                }}
                            />
                        </FormControl>
                        {/* Fee Type dropdown - full width */}
                        <FormControl fullWidth sx={{ mb: 2 }}>
                            <InputLabel>Fee Type</InputLabel>
                            <Select
                                label="Fee Type"
                                value={selectedItemForVolume.fee_type || 'FLAT'}
                                onChange={(e) => {
                                    setSelectedItemForVolume({
                                        ...selectedItemForVolume,
                                        fee_type: e.target.value
                                    });
                                }}
                                // disabled
                                helpertext="Fee type cannot be changed."
                            >
                                <MenuItem value="FLAT">Flat Fee</MenuItem>
                                <MenuItem value="PERCENTAGE">Percentage</MenuItem>
                                <MenuItem value="MIXED">Mixed (Flat + Percentage)</MenuItem>
                            </Select>
                        </FormControl>
                        {/* Fee input fields based on fee type */}
                        {selectedItemForVolume.fee_type === 'FLAT' && (
                            <TextField
                                fullWidth
                                label="Flat Rate"
                                sx={{ mb: 2 }}
                                value={selectedItemForVolume.fee_flat || ''}
                                onChange={(e) => handleFeeRateChange(e.target.value, 'flat')}
                                InputProps={{
                                    startAdornment: <Box component="span" sx={{ mr: 1 }}>USD</Box>,
                                }}
                                error={selectedItemForVolume.fee_type === 'FLAT' && rateBelowMinimumWarning !== ''}
                                helperText={selectedItemForVolume.fee_type === 'FLAT' ? rateBelowMinimumWarning : ''}
                            />
                        )}

                        {selectedItemForVolume.fee_type === 'PERCENTAGE' && (
                            <TextField
                                fullWidth
                                label="Percentage Rate"
                                sx={{ mb: 2 }}
                                value={selectedItemForVolume.fee_percentage || ''}
                                onChange={(e) => handleFeeRateChange(e.target.value, 'percentage')}
                                InputProps={{
                                    endAdornment: <Box component="span" sx={{ ml: 1 }}>%</Box>,
                                }}
                                error={selectedItemForVolume.fee_type === 'PERCENTAGE' && rateBelowMinimumWarning !== ''}
                                helperText={selectedItemForVolume.fee_type === 'PERCENTAGE' ? rateBelowMinimumWarning : ''}
                            />
                        )}

                        {selectedItemForVolume.fee_type === 'MIXED' && (
                            <>
                                <TextField
                                    fullWidth
                                    label="Flat Rate"
                                    sx={{ mb: 2 }}
                                    value={selectedItemForVolume.fee_flat || ''}
                                    onChange={(e) => handleFeeRateChange(e.target.value, 'flat')}
                                    InputProps={{
                                        startAdornment: <Box component="span" sx={{ mr: 1 }}>USD</Box>,
                                    }}
                                    error={rateBelowMinimumWarning !== '' && rateBelowMinimumWarning.includes('flat component')}
                                    helperText={rateBelowMinimumWarning !== '' && rateBelowMinimumWarning.includes('flat component') ? rateBelowMinimumWarning : ''}
                                />
                                <TextField
                                    fullWidth
                                    label="Percentage Rate"
                                    sx={{ mb: 2 }}
                                    value={selectedItemForVolume.fee_percentage || ''}
                                    onChange={(e) => handleFeeRateChange(e.target.value, 'percentage')}
                                    InputProps={{
                                        endAdornment: <Box component="span" sx={{ ml: 1 }}>%</Box>,
                                    }}
                                />
                            </>
                        )}

                        {filteredEditSlabs.length > 0 ? (
                            <Box sx={{ mt: 2 }}>
                                <Typography variant="subtitle1" gutterBottom>
                                    Available rate for your volume
                                </Typography>
                                <Grid container spacing={2}>
                                    {filteredEditSlabs.map((slab, index) => {
                                        const negotiationLevels = formatNegotiationLevels(slab);
                                        const mixedFeeInfo = formatMixedFeeInfo(slab);
                                        console.log("Enriched slab: index", index, slab);

                                        return (
                                            <Grid item xs={12} key={index}>
                                                <Paper
                                                    elevation={3}
                                                    sx={{
                                                        p: 3,
                                                        backgroundColor: 'white',
                                                        border: '1px solid #e0e0e0',
                                                        borderLeft: '4px solid #4caf50' // Highlight for slab match
                                                    }}
                                                >
                                                    {/* Header Section */}
                                                    <Box sx={{ mb: 2 }}>
                                                        <Grid container spacing={2} alignItems="center">
                                                            <Grid item xs={12} md={8}>
                                                                <Typography variant="h6" color="primary">
                                                                    Rate Configuration Details
                                                                </Typography>
                                                            </Grid>
                                                            <Grid item xs={12} md={4}>
                                                                <Chip
                                                                    label={slab.status || 'ACTIVE'}
                                                                    color={slab.status === 'ACTIVE' ? 'success' : 'default'}
                                                                    size="small"
                                                                />
                                                                {slab.custom_fee_enable === 1 && (
                                                                    <Chip
                                                                        label="MIXED FEE"
                                                                        color="warning"
                                                                        size="small"
                                                                        sx={{ ml: 1 }}
                                                                    />
                                                                )}
                                                            </Grid>
                                                        </Grid>
                                                    </Box>

                                                    <Divider sx={{ mb: 2 }} />

                                                    {/* Basic Information */}
                                                    <Grid container spacing={2} sx={{ mb: 2 }}>
                                                        <Grid item xs={12} md={6}>
                                                            <Typography variant="subtitle2" color="textSecondary">Volume Range</Typography>
                                                            <Typography variant="body1">
                                                                {slab.min_value || '0.00'} - {slab.max_value === 999999999 ? '∞' : (slab.max_value || 'Unlimited')}
                                                                {slab.velocity_type === 1 ? ' Transactions' : ' USD'}
                                                            </Typography>
                                                        </Grid>
                                                        <Grid item xs={12} md={6}>
                                                            <Typography variant="subtitle2" color="textSecondary">Fee Calculation Mode</Typography>
                                                            <Typography variant="body1" sx={{ textTransform: 'capitalize' }}>
                                                                {slab.calculation_mode || 'Flat'}
                                                            </Typography>
                                                        </Grid>
                                                        <Grid item xs={12} md={6}>
                                                            <Typography variant="subtitle2" color="textSecondary">Payout Currency</Typography>
                                                            <Typography variant="body1">{slab.payout_currency || 'USD'}</Typography>
                                                        </Grid>
                                                        <Grid item xs={12} md={6}>
                                                            <Typography variant="subtitle2" color="textSecondary">Payout Method</Typography>
                                                            <Typography variant="body1">{getPayoutMethodName(slab.payout_type)}</Typography>
                                                        </Grid>
                                                        <Grid item xs={12} md={6}>
                                                            <Typography variant="subtitle2" color="textSecondary">Coverage</Typography>
                                                            <Typography variant="body1">{slab.coverage || 'Standard'}</Typography>
                                                        </Grid>
                                                        <Grid item xs={12} md={6}>
                                                            <Typography variant="subtitle2" color="textSecondary">Speed</Typography>
                                                            <Typography variant="body1">{slab.speed || 'Standard'}</Typography>
                                                        </Grid>
                                                        {slab.transaction_type && (
                                                            <Grid item xs={12} md={6}>
                                                                <Typography variant="subtitle2" color="textSecondary">Transaction Type</Typography>
                                                                <Typography variant="body1">{slab.transaction_type == 1 ? "Transaction count" : "Amount based"}</Typography>
                                                            </Grid>
                                                        )}
                                                    </Grid>

                                                    {/* Velocity Information */}
                                                    <Grid container spacing={2} sx={{ mb: 2 }}>
                                                        <Grid item xs={12} md={6}>
                                                            <Typography variant="subtitle2" color="textSecondary">Velocity Type</Typography>
                                                            <Typography variant="body1">
                                                                {getVelocityTypeLabel(slab.velocity_type)}
                                                            </Typography>
                                                        </Grid>
                                                        <Grid item xs={12} md={6}>
                                                            <Typography variant="subtitle2" color="textSecondary">Velocity Period</Typography>
                                                            <Typography variant="body1">
                                                                {getVelocityPeriodLabel(slab.velocity_period)}
                                                            </Typography>
                                                        </Grid>
                                                    </Grid>

                                                    <Divider sx={{ mb: 2 }} />

                                                    {/* Fee Structure */}
                                                    {mixedFeeInfo ? (
                                                        <Box sx={{ mb: 2 }}>
                                                            <Typography variant="subtitle2" color="textSecondary" sx={{ mb: 1 }}>
                                                                Mixed Fee Structure
                                                            </Typography>
                                                            <Grid container spacing={2}>
                                                                <Grid item xs={12} md={6}>
                                                                    <Paper variant="outlined" sx={{ p: 2, bgcolor: 'grey.50' }}>
                                                                        <Typography variant="caption" color="textSecondary">
                                                                            Primary Component ({mixedFeeInfo.primary.type})
                                                                        </Typography>
                                                                        <Typography variant="body1">
                                                                            {mixedFeeInfo.primary.type === 'percentage'
                                                                                ? `${mixedFeeInfo.primary.value}%`
                                                                                : `${mixedFeeInfo.primary.value} USD`}
                                                                        </Typography>
                                                                    </Paper>
                                                                </Grid>
                                                                <Grid item xs={12} md={6}>
                                                                    <Paper variant="outlined" sx={{ p: 2, bgcolor: 'grey.50' }}>
                                                                        <Typography variant="caption" color="textSecondary">
                                                                            Secondary Component ({mixedFeeInfo.secondary.type})
                                                                        </Typography>
                                                                        <Typography variant="body1">
                                                                            {mixedFeeInfo.secondary.type === 'percentage'
                                                                                ? `${mixedFeeInfo.secondary.value}%`
                                                                                : `${mixedFeeInfo.secondary.value} USD`}
                                                                        </Typography>
                                                                    </Paper>
                                                                </Grid>
                                                            </Grid>
                                                            <Typography variant="body2" color="primary" sx={{ mt: 1, fontWeight: 'medium' }}>
                                                                Combined Rate: {formatFeeDisplay('MIXED', mixedFeeInfo.primary.type === 'flat' ? mixedFeeInfo.primary.value : mixedFeeInfo.secondary.value, mixedFeeInfo.primary.type === 'percentage' ? mixedFeeInfo.primary.value : mixedFeeInfo.secondary.value)}
                                                            </Typography>
                                                        </Box>
                                                    ) : (
                                                        <Box sx={{ mb: 2 }}>
                                                            <Typography variant="subtitle2" color="textSecondary" sx={{ mb: 1 }}>
                                                                Current Fee Rate
                                                            </Typography>
                                                            <Paper variant="outlined" sx={{ p: 2, bgcolor: 'primary.50' }}>
                                                                <Typography variant="h6" color="primary">
                                                                    {slab.calculation_mode === 'percentage'
                                                                        ? `${parseFloat(slab.proposed_fee || 0).toFixed(2)}%`
                                                                        : slab.calculation_mode === 'flat'
                                                                            ? `${parseFloat(slab.proposed_fee || 0).toFixed(2)} USD`
                                                                            : formatFeeDisplay(
                                                                                slab.calculation_mode || 'FLAT',
                                                                                slab.fee_flat || 0,
                                                                                slab.fee_percentage || 0
                                                                            )}
                                                                </Typography>
                                                            </Paper>
                                                        </Box>
                                                    )}

                                                    {/* Negotiation Levels */}
                                                    {negotiationLevels.length > 0 && (
                                                        <Box sx={{ mb: 2 }}>
                                                            <Typography variant="subtitle2" color="textSecondary" sx={{ mb: 1 }}>
                                                                Available Negotiation Levels
                                                            </Typography>
                                                            <List dense>
                                                                {negotiationLevels.map((level, levelIndex) => (
                                                                    <ListItem key={levelIndex} sx={{ py: 0.5 }}>
                                                                        <ListItemText
                                                                            primary={
                                                                                <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
                                                                                    <Typography variant="body2" color="textSecondary">
                                                                                        {level.level}:
                                                                                    </Typography>
                                                                                    <Typography variant="body2" sx={{ fontWeight: 'medium' }}>
                                                                                        {level.value}
                                                                                    </Typography>
                                                                                </Box>
                                                                            }
                                                                        />
                                                                    </ListItem>
                                                                ))}
                                                            </List>
                                                        </Box>
                                                    )}

                                                    {/* Action Note */}
                                                    <Box sx={{ mt: 2, p: 2, bgcolor: 'success.50', borderRadius: 1 }}>
                                                        <Typography variant="body2" color="success.main">
                                                            ✅ This rate configuration matches your volume commitment and is available for selection.
                                                        </Typography>
                                                    </Box>
                                                </Paper>
                                            </Grid>
                                        );
                                    })}
                                </Grid>
                            </Box>
                        ) : hasAttemptedVolumeChange && selectedItemForVolume.committed_volume ? (
                            <Alert severity="warning" sx={{ mt: 2 }}>
                                Cannot Edit: No matching rate slabs found for the specified volume. Please adjust your volume commitment.
                            </Alert>
                        ) : availableVolumeSlabs.length > 0 ? (
                            <Alert severity="info" sx={{ mt: 2 }}>
                                Enter a volume commitment to see available rate slabs.
                            </Alert>
                        ) : null}

                        {rateBelowMinimumWarning && (
                            <Alert severity="warning" sx={{ mt: 2 }}>
                                {rateBelowMinimumWarning}
                                <Typography variant="body2" color="textSecondary" sx={{ mt: 1 }}>
                                    Note: Setting rates below minimum may require additional approval.
                                </Typography>
                            </Alert>
                        )}
                    </Box>
                    {error && (
                        <Alert severity="error" sx={{ mt: 2 }}>
                            {error}
                        </Alert>
                    )}
                </DialogContent>
                <DialogActions>
                    <Button onClick={() => {
                        setEditVolumeModal(false);
                        setSelectedItemForVolume({
                            id: null,
                            country_code: '',
                            payout_type: '',
                            payout_method: '',
                            committed_volume: 0,
                            fee_type: 'FLAT',
                            fee_flat: 0,
                            fee_percentage: 0,
                            volume_type: 'AMOUNT'
                        });
                        setError(null);
                    }}>
                        Cancel
                    </Button>
                    <Button
                        variant="contained"
                        color="primary"
                        onClick={() => handleSlabUpdate(selectedItemForVolume)}
                        disabled={!selectedItemForVolume.id}
                    >
                        Update Volume and Rate
                    </Button>
                </DialogActions>
            </Dialog>

            <Dialog
                open={dialog.open && dialog.type === DialogType.CONFIRM}
                onClose={() => setDialog((prev) => ({ ...prev, open: false }))}
            >
                <DialogTitle>{dialog.title}</DialogTitle>
                <DialogContent>
                    <Typography>{dialog.msg}</Typography>
                </DialogContent>
                <DialogActions>
                    <Button onClick={() => setDialog((prev) => ({ ...prev, open: false }))}>
                        Cancel
                    </Button>
                    <Button
                        variant="contained"
                        color="primary"
                        onClick={() => {
                            dialog.action();
                            setDialog((prev) => ({ ...prev, open: false }));
                        }}
                    >
                        Confirm
                    </Button>
                </DialogActions>
            </Dialog>
        </Box>
    );
};

export default ViewProposal;
import { getName } from "country-list";

/**
 * Get country name with special handling for SEPA region
 * @param {string} countryCode - The country code (e.g., 'US', 'GB', '77')
 * @returns {string} The country name or 'SEPA Region' for code '77'
 */
export const getCountryName = (countryCode) => {
    // Handle special case for SEPA region (country code 77)
    if (countryCode === '77') {
        return 'SEPA Region';
    }
    
    // Use country-list library for standard country codes
    return getName(countryCode);
};

/**
 * Format country display with code in parentheses
 * @param {string} countryCode - The country code
 * @returns {string} Formatted display like 'United States (US)' or 'SEPA Region (77)'
 */
export const formatCountryDisplay = (countryCode) => {
    const countryName = getCountryName(countryCode);
    return `${countryName} (${countryCode})`;
};
this is pms module in this add proposal page there are three steps i think select countries partners and select payouts 
after selecting that in step 2 configure services section there is a fee level dropdown with options 
proposed fee
negotiation level 1,2 
minimum fee 
if the user have selected minimum fee as 7 and entering fee less than 7 red highlight will be shown but after giving less than the minimum fee and giving submit proposal after guving next 
there a reason comment box is supposed to come previously they are saying it was coming but now its not coming 
